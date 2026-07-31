# Troubleshooting Ansible
Red Hat Enterprise Linux 10 includes `ansible-core` 2.16 for managing RHEL 9 and RHEL 10 systems. Effective troubleshooting separates four layers: playbook logic, inventory data, connection transport, and privilege escalation. Each layer has distinct tests, failure modes, and security risks.
## Safe preflight checks
`ansible-playbook --syntax-check playbook.yml` detects syntax problems, but it cannot validate runtime data or task logic. Check mode simulates supported changes:

```shell
ansible-playbook --check playbook.yml
```

A module must support check mode for the simulation to be useful. Tasks that register results from skipped changes can also produce misleading later conditions. `check_mode: false` forces a task to run normally during a check-mode execution, so administrators should reserve it for safe diagnostics.

Diff mode displays proposed file and template changes:

```shell
ansible-playbook --check --diff playbook.yml
```

Diff output can expose credentials or other confidential content. Administrators should restrict access to the output, limit the affected hosts, and set `diff: false` on sensitive tasks.

Other useful controls include `--list-tasks`, `--step`, and `--start-at-task`. Dynamic includes can hide their child tasks from `--list-tasks`, and `--start-at-task` cannot target those hidden tasks.

Before a broad run, `--list-hosts` confirms the host pattern without executing tasks. `--limit` then narrows a test to one host or a small group. This sequence prevents an inventory mistake from turning a diagnostic run into an unintended fleet-wide change. Check mode remains a simulation rather than a transaction, so a successful check cannot guarantee a successful live run.
## Results and evidence
The play recap distinguishes execution outcomes:

| State | Meaning |
|---|---|
| `ok` | The task succeeded without reporting a change |
| `changed` | The task succeeded and reported a change |
| `unreachable` | Ansible could not use the configured connection, so it removed the host from the active run |
| `failed` | A module, command, or custom condition reported failure |
| `skipped` | A condition, tag selection, or other rule prevented execution |
| `rescued` | A `rescue` block handled a failed task |
| `ignored` | An ignored task failed, but execution continued |

A task failure normally stops later tasks on that host while other hosts continue. Higher verbosity, from `-v` through `-vvvv`, adds diagnostic detail. `-vvv` often reveals connection and interpreter decisions. Verbose output can also reveal secrets.

Ansible writes output to standard output by default. The `log_path` setting or `ANSIBLE_LOG_PATH` environment variable can retain a log. Teams should protect and rotate that log. `no_log: true` suppresses task results, but it does not protect secrets printed by an explicit `debug` task. Only one stdout callback can control console output at a time.

Useful evidence includes the first failing task, the affected host, the module arguments after variable resolution, the return code, standard output, standard error, and registered data. Administrators should investigate the first substantive failure rather than later symptoms. They should also compare successful and failing hosts because differences in inventory variables, packages, permissions, and network paths often isolate the cause.
## Accurate failure and change reporting
Automation should encode the accepted result instead of hiding unexpected errors. `failed_when` and `changed_when` accept raw Jinja expressions without `{{ }}`:

```yaml
- name: Check the Apache service
  ansible.builtin.command: systemctl is-active httpd
  register: httpd_state
  changed_when: false
  failed_when: httpd_state.rc not in [0, 3]
```

`ignore_errors` applies only when a task returns a failed state. It does not cover syntax errors, undefined variables, or connection failures. `ignore_unreachable` controls unreachable hosts separately.

Blocks support structured recovery with `block`, `rescue`, and `always`. Administrators can also use `any_errors_fatal` to stop all hosts, `max_fail_percentage` to enforce a failure threshold, and `force_handlers` to run notified handlers after a later task failure.

A `rescue` section handles tasks that return failure within its block. It does not repair an unreachable host or an invalid task definition. Handlers normally do not run on a host after a later task fails, even if an earlier task notified them. `force_handlers` changes that behaviour, but a handler can still fail when the host becomes unreachable.
## Diagnostic modules
Fully qualified collection names make task origin clear.

| Module | Diagnostic purpose |
|---|---|
| `ansible.builtin.debug` | Displays selected variables or messages, excluding secrets |
| `ansible.builtin.uri` | Checks HTTP endpoints, status codes, response content, and TLS behaviour |
| `ansible.builtin.stat` | Collects file, link, ownership, permission, and checksum data |
| `ansible.builtin.assert` | Verifies one or more conditions and reports a precise failure message |
| `ansible.builtin.fail` | Stops a task deliberately when a condition identifies an invalid state |
| `ansible.builtin.command` | Runs a focused diagnostic command with `changed_when: false` |

An assertion should match its message exactly. A condition that accepts values from 1 through 100 should not claim that zero is valid. File-size tests should also use correct units rather than confusing bytes with megabytes.

Registered results preserve module-specific fields for later inspection. An HTTP check, for example, can register the response from `ansible.builtin.uri`, require an accepted status code, and assert expected content. TLS certificate validation should remain enabled unless an authorised diagnostic test requires a temporary exception. A `debug` task should display only the field needed to test the hypothesis.
## Focused runs with tags
Tags allow administrators to run or skip selected work with `--tags`, `--skip-tags`, and `--list-tags`. The special tag `always` runs unless it is explicitly skipped. The special tag `never` skips a task unless the command requests `never` or another tag attached to that task.

```yaml
- name: Show detailed service data
  ansible.builtin.debug:
    var: ansible_facts.services
  tags:
    - never
    - debug
```

`--tags debug` runs this diagnostic task. `--tags all,debug` runs regular tasks and the diagnostic task.

Tags on plays, blocks, roles, and static imports pass to their contained tasks. Tags can also label dynamic `include_tasks` and `include_role` statements, but they do not pass to included tasks by default. The `apply` keyword, a tagged block, or a static import provides inheritance when required.
## Connectivity and privilege escalation
`ansible.builtin.ping` does not send an ICMP echo request. It verifies that Ansible can log in through the configured connection and execute usable Python on the managed node. It does not test privilege escalation unless the command enables `become`, and it does not validate an application service.

Administrators should diagnose connectivity in layers:
1. Inspect the compiled host data with `ansible-inventory -i inventory --host web1`.
2. Verify name resolution, routing, and the configured SSH port.
3. Test direct SSH with the same user, key, port, and jump-host settings.
4. Run `ansible web1 -i inventory -m ansible.builtin.ping -vvv`.
5. Test escalation separately with `ansible web1 -i inventory -m ansible.builtin.command -a id --become -K`.

`ansible_host` can map an inventory alias to a hostname or address. RHEL 10 uses automatic Python interpreter discovery by default, so `ansible_python_interpreter` should be set only when discovery selects an unsuitable interpreter.

`become: true` enables privilege escalation. `become_user` selects the target account and defaults to `root`, but setting it alone does not enable escalation. The `-K` option requests a privilege-escalation password, while `-k` requests an SSH login password. Separating login tests from escalation tests quickly identifies whether authentication, Python discovery, or authorisation caused the failure.

Common connection faults include a wrong host or port, failed name resolution, a blocked route, a stale host key, an unsuitable SSH identity, an incorrect remote user, and a missing jump-host configuration. A successful direct SSH session does not prove that Ansible used the same settings. Verbose Ansible output reveals the effective user, connection command, and interpreter choice needed for comparison.