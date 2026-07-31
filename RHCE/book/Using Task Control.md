# Using Task Control
Ansible controls task execution through loops, conditions, handlers, result tests, and blocks. These features let a playbook apply the same operation to several values, select tasks for each host, react to genuine changes, and recover from failures.

RHEL 10 supplies `ansible-core` 2.16 for control nodes that use RHEL System Roles. It uses DNF for software management and systemd for services. Playbooks should therefore favour `ansible.builtin.dnf`, `ansible.builtin.systemd_service`, native Boolean values such as `true`, and fully qualified collection names. The `yum` name remains an alias for the DNF module, but `ansible.builtin.dnf` states the intended package manager clearly.
## Loops and list data
A loop runs one task once for each value in a list. During each iteration, Ansible assigns the current value to `item`. The `loop` keyword sits at the task level, alongside the module call.

Modules that accept a list should receive the complete list directly. A single DNF transaction is more efficient than one transaction per package. Modules that accept one value, such as a service module acting on one unit, require a loop.

```yaml
---
- name: Configure RHEL 10 web hosts
  hosts: web
  become: true
  vars:
    web_packages:
      - httpd
      - mod_ssl
    web_services:
      - httpd

  tasks:
    - name: Install web packages
      ansible.builtin.dnf:
        name: "{{ web_packages }}"
        state: present

    - name: Enable and start web services
      ansible.builtin.systemd_service:
        name: "{{ item }}"
        enabled: true
        state: started
      loop: "{{ web_services }}"
```

Lists of dictionaries carry several values for each iteration. A user task, for example, can read `item.name`, `item.shell`, and `item.groups`. A dictionary that is not already a list can pass through the `dict2items` filter before iteration. `loop_control.loop_var` replaces `item` with a clearer name and prevents collisions in included tasks. `loop_control.label` limits verbose output to a useful identifier.

The older `with_*` forms use lookup plugins. `loop` provides the clearest syntax for ordinary lists, while specialised lookups can use `query()` to return list data.

When a looped task registers its result, the registered variable contains a `results` list. Each element records the outcome for one item, including fields such as `item`, `changed`, `failed`, `rc`, `stdout`, and `stderr`, where the module supplies them. A later task can loop over that list and select entries by status.

This package audit treats both "installed" and "not installed" as expected query results. It then reports only missing packages:

```yaml
- name: Query required packages
  ansible.builtin.command:
    argv:
      - rpm
      - -q
      - "{{ package_name }}"
  loop: "{{ web_packages }}"
  loop_control:
    loop_var: package_name
    label: "{{ package_name }}"
  register: package_query
  changed_when: false
  failed_when: package_query.rc not in [0, 1]

- name: Report a missing package
  ansible.builtin.debug:
    msg: "{{ item.package_name }} is not installed"
  loop: "{{ package_query.results }}"
  loop_control:
    label: "{{ item.package_name }}"
  when: item.rc == 1
```

The custom loop variable appears in every registered result as `package_name`. Restricting accepted return codes prevents an expected negative query from stopping the play while preserving genuine command failures.
## Conditional execution
The `when` keyword evaluates a raw Jinja expression for each host. It does not use `{{ }}` delimiters. Literal strings still need YAML quoting. An unquoted name does not become a number. Jinja treats it as an identifier, which can produce an undefined-variable error.

Common tests include:
- `variable is defined` and `variable is not defined`
- `value in values`
- `feature_enabled` and `not feature_enabled`
- `result.rc == 0`
- `result.stdout is search('text')`
- Numeric comparisons such as `available_bytes >= required_bytes`

A YAML list under `when` combines conditions with logical AND. A single expression can use `and`, `or`, and parentheses when the logic requires alternatives. Filters should convert values before numeric comparison because many facts and external variables arrive as strings.

Operator precedence can obscure a long expression. Parentheses should group each alternative, especially when an expression mixes `and` with `or`. A folded YAML scalar marked with `>` can spread that expression across lines without changing its logical value. Each condition should evaluate to a Boolean rather than a quoted string that resembles one.

This task runs only on RHEL 10 hosts:

```yaml
- name: Install the Apache HTTP Server on RHEL 10
  ansible.builtin.dnf:
    name: httpd
    state: present
  when:
    - ansible_facts['distribution'] == 'RedHat'
    - ansible_facts['distribution_major_version'] | int == 10
```

Facts must be gathered or supplied before a condition can use them. `ansible_facts['os_family']`, `ansible_facts['distribution']`, and `ansible_facts['distribution_major_version']` support operating-system decisions. Tests should use the narrowest fact that represents the requirement. An exact RHEL 10 requirement should test the distribution and major version, rather than accepting every member of the Red Hat family.

`when` can also filter loop iterations. For example, a task can loop over `ansible_facts['mounts']` and run only when `item.mount == '/boot'` and `item.size_available` exceeds the required byte count. Ansible reports the other iterations as skipped.

Registered results support decisions based on earlier work. Membership tests such as `'lisa' in account_data.stdout` are clearer than comparing the result of Python's `find()` method with `-1`. Interactive `vars_prompt` input can set a variable, but unattended automation should obtain values from inventory, variable files, a survey, or another controlled input source.
## Handlers
A handler performs an operation only after a notifying task reports `changed`. A successful task that reports `ok` does not notify its handlers. This rule prevents unnecessary service restarts and other disruptive actions.

```yaml
- name: Manage the Apache HTTP Server
  hosts: web
  become: true
  tasks:
    - name: Deploy the web configuration
      ansible.builtin.template:
        src: httpd.conf.j2
        dest: /etc/httpd/conf/httpd.conf
        owner: root
        group: root
        mode: '0644'
      notify: Restart httpd

  handlers:
    - name: Restart httpd
      ansible.builtin.systemd_service:
        name: httpd
        state: restarted
```

Ansible queues a notified handler and normally runs it after each relevant section: `pre_tasks`, `roles` or `tasks`, and `post_tasks`. Repeated notifications queue the same handler once. Handlers run in their definition order, not their notification order. `ansible.builtin.meta: flush_handlers` runs queued handlers earlier when subsequent tasks depend on the changed service state.

A later failure on a host normally suppresses that host's queued handlers. `force_handlers: true` at play level runs notified handlers despite later task failures, although unreachable hosts can still prevent execution. This option does not turn an unchanged task into a notification.

When a looped task notifies handlers, any changed iteration marks the entire task as changed and triggers every notification attached to that task. Separate tasks or carefully selected handler topics avoid restarting unrelated services. The `listen` keyword lets several uniquely named handlers subscribe to one topic, which separates the notification interface from handler names.
## Failures and task status
Ansible normally stops later tasks on the host where a task fails and continues on other hosts. `any_errors_fatal: true` stops the play across the current batch after a failure. `ignore_errors: true` lets execution continue after a task returns `failed`, but it does not cover unreachable hosts, invalid task definitions, syntax errors, or undefined variables. `ignore_unreachable` controls unreachable-host handling separately.

Task status affects reporting, conditions, and handlers:
- `ok` means that the task succeeded without changing the host.
- `changed` means that the task succeeded and changed the host.
- `failed` means that the module or a custom test declared failure.
- `unreachable` means that Ansible could not communicate with the host.
- `skipped` means that a condition prevented execution.

Command and shell tasks usually report `changed` whenever they run because Ansible cannot infer their effect. A read-only command should set `changed_when: false`. `failed_when` can reject output or return codes that the module would otherwise accept. Both keywords take raw Jinja expressions without `{{ }}`.

```yaml
- name: Validate the Apache configuration
  ansible.builtin.command: /usr/sbin/httpd -t
  register: httpd_check
  changed_when: false
  failed_when: httpd_check.rc != 0
```

A list under `failed_when` combines tests with AND. A single string containing `or` is required when any one test should cause failure. The `ansible.builtin.fail` module provides a deliberate failure with a clear message, usually guarded by `when`.

Purpose-built modules remain safer and more idempotent than shell commands. `ansible.builtin.file` should remove or create files, `ansible.builtin.dnf` should manage packages, `ansible.builtin.systemd_service` should manage units, and `ansible.builtin.reboot` should reboot a host.

Error policy should reflect the required scope. A local optional probe can use an explicit `failed_when` rule. A non-critical operation may use `ignore_errors` when later tasks can safely continue. A fleet-wide invariant can use `any_errors_fatal`. Broadly ignoring failures can leave hosts in different states and can conceal the original fault.
## Blocks and recovery
A block groups tasks and applies shared directives such as `when`, `become`, and `any_errors_fatal`. A loop cannot attach directly to a block. A repeated task group should move into a file and use `include_tasks` with a loop.

Error-handling blocks contain up to three parts:
- `block` contains the primary tasks.
- `rescue` runs after a task in `block` returns `failed`.
- `always` runs after the block outcome, whether the primary tasks succeed or fail.

Invalid task definitions and unreachable hosts do not trigger the block's `rescue` or `always` sections. A successful rescue allows later tasks to continue and records the event as rescued. The `ansible_failed_task` and `ansible_failed_result` variables expose the failed task and its result inside `rescue`.

Blocks work best for a coherent transaction, such as deploying a configuration, validating it, restoring a backup after failure, and recording the outcome. They do not replace idempotent modules or careful failure tests. Clear task names, explicit conditions, and accurate status reporting keep recovery paths understandable.

A recovery path should restore a known valid state before allowing queued handlers to run. When a rescue task restores an earlier configuration, `ansible.builtin.meta: flush_handlers` can apply that restored configuration before later work proceeds. If recovery cannot guarantee a valid state, `ansible.builtin.fail` should stop that host with a specific diagnostic message.