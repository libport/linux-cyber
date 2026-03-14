# Using Task Control
Ansible controls repeated work, conditional execution, error handling and service restarts through loops, `when`, registered results, handlers and blocks. These features keep playbooks concise while preserving predictable behaviour across hosts. They also reduce duplication, make intent easier to read and support idempotent task design.
## Loops
`loop` repeats a task for each item in a list. Modules that already accept a list should receive the list directly because one task run is simpler and usually faster than repeated runs. Package modules such as `yum`, `dnf`, `apt` and the generic `package` module commonly support list input. Modules such as `service` act on one object at a time, so a loop fits naturally.

```yaml
- name: Start services
  ansible.builtin.service:
    name: "{{ item }}"
    state: started
    enabled: true
  loop:
    - vsftpd
    - httpd
    - smb
```

A playbook can define loop data inside a task, in `vars`, in a variables file or in host and group variables. Externalising the list separates fixed task logic from site-specific data and improves reuse.

Loops also work with a list of dictionaries. Each iteration can address fields such as `item.username`, `item.groups` or `item.shell`. This pattern suits user creation, package state matrices and similar structured data. A plain dictionary does not loop directly, so Ansible converts it first with `dict2items`.

Legacy `with_*` syntax still works, but `loop` is the standard choice for simple iteration. `with_items` is not formally deprecated in current Ansible documentation. It remains valid, though `loop` is clearer and fits newer examples. Migration also needs care because `with_items` flattens one level implicitly while `loop` does not. A direct replacement can therefore change the data passed to the module.
## Conditional execution with `when`
`when` runs a task only when an expression evaluates to true. The expression is raw Jinja2, so it does not use `{{ }}`. A task can test facts, variables, return codes, list membership and numeric thresholds. That makes `when` the central control mechanism for host-specific execution.

Common patterns include these checks:
- variable existence with `is defined` or `is not defined`
- Boolean evaluation of true and false values
- string comparison with `==` and `!=`
- numeric comparison with `<`, `<=`, `>` and `>=`
- list membership with `in`

```yaml
- name: Install Apache on Red Hat systems
  ansible.builtin.yum:
    name: httpd
    state: latest
  when: ansible_facts['os_family'] == "RedHat"
```

A playbook can combine conditions with `and`, `or` and parentheses. YAML folded syntax with `>` lets long expressions remain readable without changing their logic. For mixed operating systems, memory thresholds or mount-point checks, combined conditions keep one task precise instead of scattering logic across multiple near-duplicate tasks.

When `when` and `loop` appear together, Ansible evaluates the condition for each item separately. That behaviour allows filtering inside the loop without pre-processing the list. A loop over mount facts, for instance, can update a kernel only when the current item represents `/boot` and that mount has sufficient free space.
## Registered results and task status
`register` stores task output in a variable so later tasks can react to it. Registered data often includes `stdout`, `stderr`, `rc`, `changed` and, for loops, a `results` list. That structure turns command output into explicit control flow.

A common pattern checks the state of a service or command and runs a task only when the command succeeds.

```yaml
- name: Check whether httpd is active
  ansible.builtin.command: systemctl is-active httpd
  register: result
  ignore_errors: true

- name: Restart sshd only when httpd is active
  ansible.builtin.service:
    name: sshd
    state: restarted
  when: result.rc == 0
```

String inspection can refine that decision further. A playbook can search `stdout` for a token and branch only when the expected text appears. This approach works, but return codes usually communicate intent more clearly and remain less brittle than free-text matching.

`ignore_errors` allows the play to continue after a task returns `failed`. It does not suppress syntax errors, undefined variables, unreachable hosts or other failures that prevent the task from running normally.

`failed_when` redefines what failure means. This matters when a command exits successfully but its output still signals a bad result. `changed_when` redefines what counts as change. This prevents harmless commands such as `date` from reporting a change and accidentally notifying handlers. These two controls keep play recap data meaningful and stop false positives from rippling through later logic.

These conditionals also use raw Jinja2 expressions without `{{ }}`.
## Handlers
Handlers run at the end of a play after a task reports `changed` and sends a notification. They suit actions such as restarting a service after a configuration file changes.

```yaml
tasks:
  - name: Deploy index page
    ansible.builtin.copy:
      src: /tmp/index.html
      dest: /var/www/html/index.html
    notify: restart_web

handlers:
  - name: restart_web
    ansible.builtin.service:
      name: httpd
      state: restarted
```

A handler does not run merely because a notifying task succeeds. It runs only when that task changes the target state. If the task later reports `ok`, the handler stays idle. This behaviour preserves idempotence.

Later task failure matters as well. By default, if a task notifies a handler and another task fails later in the same play, the handler does not run on that host. `force_handlers: true` changes that behaviour and forces notified handlers to run unless the host becomes unreachable or another hard failure prevents execution. This distinction matters because forcing handlers is not the same as ignoring task errors. `ignore_errors` lets the play continue, while `force_handlers` specifically preserves notified handlers after failure.

Handlers execute in the order defined in the `handlers` section, not in the order of notification. A task can notify more than one handler, but the handler names must match exactly.
## Blocks, rescue and always
A block groups related tasks so shared directives can apply once. `when`, `become`, `ignore_errors` and `any_errors_fatal` can sit on the block and flow to each enclosed task. A block itself cannot take `loop`.

```yaml
- name: Manage web content
  block:
    - name: Remove old file
      ansible.builtin.file:
        path: /var/www/html/index.html
        state: absent
    - name: Write log entry
      ansible.builtin.command: logger cleanup complete
  when: ansible_facts['distribution'] == "CentOS"
```

`rescue` defines recovery tasks that run after a task in the main block fails. `always` defines tasks that run whether the block succeeds or fails. This pattern gives a playbook structured error handling and can replace brittle, ad hoc clean-up steps.

A rescue section responds only to tasks that return `failed`. It does not catch invalid task definitions or unreachable hosts. If a rescue task succeeds, Ansible continues the play and records the failure as rescued. In practice, that means the play can recover from an operational error without pretending that nothing happened.

`any_errors_fatal: true` stops the play across all hosts after a fatal task fails in the current batch. This suits operations that require all hosts to stay in lockstep, such as coordinated maintenance or staged rollouts with strict dependency order.
## Operational guidance
Purpose-built modules should take priority over raw `command` or `shell` calls whenever Ansible already models the desired state. Removing a file with the `file` module is clearer and more idempotent than calling `rm`. Creating a file with module support is usually safer than calling `touch`. Module choice improves reporting, reduces warnings and keeps playbooks closer to declarative state management.
## Practical rules
- Prefer a module parameter that accepts a list over looping one item at a time.
- Keep loop data in variables when the environment is likely to change.
- Use `register` with return codes and output fields to make task flow explicit.
- Use `failed_when` and `changed_when` to align Ansible reporting with real outcomes.
- Notify handlers only from tasks that genuinely alter state.
- Use `force_handlers` when a configuration change must still trigger its follow-up action after later failure.
- Use blocks to group related tasks and to attach `rescue` and `always` logic.
- Prefer purpose-built modules over raw `command` or `shell` invocations whenever an Ansible module already models the desired state.