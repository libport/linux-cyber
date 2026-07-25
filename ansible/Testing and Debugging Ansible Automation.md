# Testing and Debugging Ansible Automation
Reliable Ansible automation depends on fast feedback, controlled execution, explicit validation, useful diagnostics, and automated quality checks. Teams should test on a narrow scope, inspect predicted changes, validate assumptions inside playbooks, and enforce consistent standards before production deployment.

Effective use requires familiarity with Linux or Unix command-line tools, inventories, playbooks, roles, modules, variables, facts, and Git.
## Layered assurance
Testing works best as a sequence of increasingly realistic checks. Each stage answers a different question and prevents cheap defects from reaching expensive environments.

1. Syntax checking confirms that Ansible can parse the playbook.
2. Static analysis checks syntax, style, risky practices, and deprecated features across the repository.
3. Check mode and diff mode estimate behaviour on a small, representative target.
4. A normal run in a disposable or non-production environment tests module behaviour, dependencies, permissions, conditions, and handlers.
5. Assertions and service checks verify preconditions and outcomes during execution.
6. A staged production rollout limits exposure while monitoring failures, changes, and service health.

Idempotence needs explicit attention throughout this sequence. After a successful normal run, a second run should usually report no changes. Unexpected changes on every run can reveal unstable input, non-idempotent commands, timestamps embedded in managed files, or modules used without suitable guards. Some operations legitimately change on every run, but teams should isolate and explain them.

Test data also needs variation. A playbook that succeeds on one clean host may fail on a host with an older operating system, a different package state, limited disk space, an interrupted prior deployment, or missing network access. Representative test groups should cover the important differences in the managed population.
## Command-line testing and diagnosis
### Validate syntax
`ansible-playbook --syntax-check` parses a playbook without running it:

```bash
ansible-playbook site.yml --syntax-check
```

A successful check normally returns exit status 0, which suits automated pipelines. A non-zero status indicates an error. Shell globbing can check several playbooks, although a dedicated linting configuration gives repositories more consistent coverage.

Syntax checking confirms that Ansible can parse the playbook and recognise its structure. It does not prove that variables exist at runtime, module arguments are valid for every host, conditions express the intended logic, external systems are reachable, or tasks produce the required result. Teams must combine it with other tests.
### Reduce the execution scope
`--limit` restricts a run to hosts or groups that match an inventory pattern:

```bash
ansible-playbook site.yml --limit app01
ansible-playbook site.yml --limit webservers
```

Testing one representative host shortens feedback cycles and reduces the blast radius. A sound rollout expands progressively from a development host to a small group, a larger cohort, and then the full target population. The selected host still needs realistic variables, operating-system versions, services, and dependencies.

Tags restrict execution to labelled tasks, blocks, plays, roles, or imported content:

```bash
ansible-playbook database-upgrade.yml --tags database
ansible-playbook database-upgrade.yml --tags start,stop
ansible-playbook database-upgrade.yml --skip-tags disruptive
```

Tags help isolate a component in a long playbook, but they can bypass prerequisite tasks and handlers. Teams should design tag boundaries deliberately and test both tagged paths and complete runs. `--list-tags` and `--list-tasks` help inspect the selected path before execution.

`--start-at-task` can resume execution at a named task, and `--step` asks for confirmation before each task. These options help investigate long playbooks, but neither reconstructs state that earlier tasks would have created. They therefore suit controlled diagnosis, not proof that an isolated task works in a complete run.
### Predict changes
Check mode asks supporting modules to predict changes without changing remote systems:

```bash
ansible-playbook site.yml --check --limit app01
```

It can reveal drift, likely changes, and some invalid inputs. It remains a simulation. Modules that do not support check mode do nothing and report nothing, while tasks that depend on registered results may not behave normally. A task with `check_mode: false` can still change a system during a check-mode run.

Simulation cannot reproduce every external effect. Package repositories can change, APIs can return different data, commands may lack predictive behaviour, and race conditions may appear only during execution. Teams should treat a clean check-mode run as evidence that supports a deployment decision, not as a guarantee.

Diff mode shows before-and-after information for supporting modules. It can run alone to display changes actually made, or with check mode to display predicted changes:

```bash
ansible-playbook site.yml --check --diff --limit app01
```

Diff output may expose passwords, keys, configuration secrets, or personal information. Teams should limit its scope and set `diff: false` on sensitive tasks.

At task level, `check_mode: true` always simulates a supporting task, even during a normal run. `check_mode: false` forces normal execution, even when the command includes `--check`. This control can support a focused test, such as updating a package cache normally before simulating package installation. Because forced normal execution weakens the safety boundary, teams should use it sparingly and document the reason.
### Increase diagnostic detail
Each additional `-v` increases verbosity:

```bash
ansible-playbook site.yml -vvv
```

Moderate verbosity exposes task locations, returned data, configuration choices, and connection details. Connection faults may require `-vvvv`. Excessive output can obscure the original failure, so teams should begin with a useful level and increase it only when needed.

`ANSIBLE_LOG_PATH` or the corresponding `log_path` configuration setting saves controller output to a file:

```bash
ANSIBLE_LOG_PATH=./ansible.log ansible-playbook site.yml -vvv
```

Saved logs support searching, comparison, collaboration, and defect reports. They also create a security obligation. Teams should restrict file permissions and retention, avoid public uploads, redact sensitive content, and apply `no_log: true` to tasks that could expose secrets. `no_log` does not protect debug output.

`ANSIBLE_DEBUG=1` enables detailed internal diagnostics. This output targets difficult core, plugin, or module problems and can be extremely large. It should normally go to a protected file and remain disabled in routine production runs.
## Validation and control inside playbooks
### Inspect values with `ansible.builtin.debug`
The debug module prints a message or a variable during execution. The `msg` and `var` parameters are mutually exclusive, and `verbosity` controls the minimum command-line verbosity required to display the output.

```yaml
- name: Show available space at higher verbosity
  ansible.builtin.debug:
    var: ansible_mounts
    verbosity: 2
```

The `var` parameter already evaluates its value in a Jinja context, so it normally takes a variable name without `{{ }}` delimiters. Debug tasks can inspect gathered facts, registered results, filters, and conditional inputs during development. Production debug output should communicate only information that operators need and should never reveal credentials or other sensitive values.

A useful fact-finding pattern begins with a restricted host, prints the relevant fact structure, filters it to the required record, extracts a value, and assigns a clearly named fact. The final task then uses the derived value in a `when` condition or assertion. For example, automation can select the root mount from `ansible_mounts`, extract its available bytes, convert the result to an integer, and require a minimum capacity before updating packages. Temporary debug tasks should be removed or assigned an appropriate verbosity before release.
### Assert preconditions and outcomes
`ansible.builtin.assert` evaluates every expression supplied to `that`. A false expression fails the task for the current host. Custom `fail_msg` and `success_msg` values should state the failed condition and include safe, relevant values.

```yaml
- name: Require sufficient free space
  ansible.builtin.assert:
    that:
      - available_bytes | int > 5000000000
    fail_msg: Insufficient free space for the package update
    success_msg: Free-space check passed
```

Assertions can reject invalid VLAN identifiers, unsupported operating systems, unsafe input ranges, unhealthy services, and incomplete post-change states. They turn hidden assumptions into executable safeguards.

`ansible.builtin.fail` stops the current host deliberately when a custom condition becomes true. It works well after a command or API call whose return code cannot express the operational requirement. For example, automation can parse a latency measurement and fail when the result exceeds an approved threshold. Prefer a purpose-built module or structured return data to fragile parsing of human-readable command output.

Assertions suit one or more conditions that must all pass. A conditional fail task suits the inverse form, where one defined state must stop execution. Both operate per host by default, allowing other hosts to continue according to Ansible's failure settings. Teams should choose failure messages that identify the host, failed condition, observed safe value, and expected range without exposing secrets.
### Change Ansible's execution state
`ansible.builtin.meta` controls Ansible itself rather than a managed resource. Common actions include:
- `flush_handlers` runs notified handlers immediately.
- `refresh_inventory` reloads dynamic inventory data, but it does not replace hosts in the current play automatically.
- `clear_facts` clears gathered facts and persistent cached facts for selected hosts.
- `end_host` ends the current play for one host without recording a failure.
- `end_play` ends the current play for all hosts without recording failures.
- `reset_connection` interrupts a persistent connection so Ansible can establish it again.

These actions alter control flow and can surprise maintainers. Clear task names, narrow conditions, and focused tests should accompany them.

Handlers normally run at defined points, including the end of a play section. `flush_handlers` is useful when a service must restart before a later validation task, but an early restart can alter subsequent tasks. `reset_connection` becomes valuable after changing connection properties or group membership that affects the remote session.
### Wait for observable readiness
`ansible.builtin.wait_for` pauses until a condition succeeds or a timeout expires. It can wait for a file to appear or disappear, a regular expression to appear in a file, a TCP port to open or close, or active TCP connections to drain. It can also provide a fixed delay, although an observable readiness check is more reliable than an assumed duration.

```yaml
- name: Wait for the web service
  ansible.builtin.wait_for:
    port: 443
    state: started
    timeout: 30
```

The module does not perform a UDP health check. For application-level assurance, teams should use a module that validates the protocol or endpoint, such as an HTTP request, rather than treating an open TCP port as proof of service health.

Timeouts should reflect normal startup variation while still exposing faults quickly. A short timeout may create intermittent failures on busy hosts, while a long timeout delays detection. File and port checks should identify the host from which Ansible observes the condition, because execution on a managed host and delegation to the controller can produce different network views.
## Interactive task debugging
Ansible's task debugger pauses execution at a selected task and provides the task, host, result, arguments, and variables in the runtime context. It can correct an argument or variable, rerun the task, and continue without replaying the entire playbook.

The `debugger` keyword applies to a play, role, block, or task. It accepts five trigger values:

| Value | Behaviour |
| --- | --- |
| `always` | Invoke the debugger for every outcome |
| `never` | Never invoke it |
| `on_failed` | Invoke it after task failure |
| `on_unreachable` | Invoke it when the host is unreachable |
| `on_skipped` | Invoke it when Ansible skips the task |

The most specific keyword overrides broader configuration. Global alternatives include `enable_task_debugger = True` under `[defaults]` in `ansible.cfg` and `ANSIBLE_ENABLE_TASK_DEBUGGER=True`. Both invoke the debugger for failed tasks unless a more specific keyword disables it.

Seven debugger commands support the core workflow:

| Command | Purpose |
| --- | --- |
| `p expression` | Print task, host, result, argument, or variable data |
| `task.args['key'] = value` | Change a module argument |
| `task_vars['key'] = value` | Change a task variable |
| `u` or `update_task` | Recreate the task after variable changes |
| `r` or `redo` | Run the current task again |
| `c` or `continue` | Continue with the next task |
| `q` or `quit` | Abort execution and leave the debugger |

Argument changes take effect when `redo` runs. Variable changes require `update_task` before `redo`. Interactive debugging can expose or modify runtime data, so teams should use it on controlled hosts and transfer confirmed fixes back into source-controlled playbooks.
## Static analysis with Ansible Lint
Ansible Lint detects syntax problems, risky patterns, deprecated features, and inconsistent style before execution. Installation in an isolated Python environment avoids dependency conflicts:

```bash
pipx install ansible-lint
ansible-lint --version
```

The tool discovers configuration in `.ansible-lint`, `.config/ansible-lint.yml`, or `.config/ansible-lint.yaml`. A repository-level file keeps local and continuous-integration behaviour consistent. Useful settings include `profile`, `exclude_paths`, `skip_list`, and `enable_list`. Profiles allow an established repository to begin with a smaller rule set and adopt stricter checks progressively.

Fully qualified collection names clarify which module or plugin a task uses and avoid collisions between collections. Naming rules improve failure output and make `--start-at-task` practical. YAML rules promote consistent Boolean values and formatting. Deprecated-feature rules warn teams before upgrades remove old behaviour. Rule availability changes between releases, so repositories should pin the Ansible Lint version and review configuration during planned upgrades.

Running `ansible-lint` reports each violation with a rule identifier, file, and line. Teams should correct violations by default. A targeted comment such as `# noqa: name[missing]` can suppress an exceptional line, while `.ansible-lint-ignore` records accepted violations by file and rule. Suppressions need a documented reason and periodic review.

Pre-commit can run Ansible Lint before Git creates a commit. A `.pre-commit-config.yaml` file pins the hook repository and revision, after which `pre-commit install` activates it. Local hooks shorten feedback cycles, while the continuous-integration pipeline remains the authoritative check for every contribution.

Together, syntax checks, restricted test runs, simulations, in-play validation, interactive diagnosis, and static analysis create layered assurance. No single technique proves correctness. Teams gain confidence by combining fast local checks with representative test environments, controlled rollouts, secure diagnostics, repeatable pipelines, and post-change verification.