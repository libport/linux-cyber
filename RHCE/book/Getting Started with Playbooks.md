# Getting Started with Playbooks
Red Hat Enterprise Linux 10 provides `ansible-core` 2.16 for supported management of RHEL 9 and RHEL 10 nodes. Playbooks express reusable configuration, deployment, and orchestration in YAML. Unlike a shell script that chains ad hoc commands, a playbook records host selection, privilege escalation, tasks, arguments, and desired state in one structured file.
## Playbook structure
A playbook is an ordered list of plays. Each play selects managed hosts with a pattern and runs tasks against them. Tasks execute from top to bottom and call modules or other Ansible actions. Multiple plays also execute in file order.

At minimum, a play selects hosts and defines work through at least one task, either directly or through a role. A descriptive `name` is optional but strongly recommended. `become: true` enables privilege escalation when administrative rights are required. Ansible gathers host facts at the start of each play by default unless `gather_facts: false` disables that step.

The `hosts` pattern belongs to the play, while the command line supplies the inventory. Operators can use `--limit` to reduce the selected hosts for a maintenance window or staged rollout. A limit intersects the play's pattern and cannot add hosts that the play did not select.

The YAML document marker `---` is conventional but optional. The closing marker `...` is also optional. Fully qualified collection names identify modules and roles without ambiguity.

```yaml
---
- name: Configure RHEL 10 web servers
  hosts: webservers
  become: true
  tasks:
    - name: Install Apache HTTP Server
      ansible.builtin.dnf:
        name: httpd
        state: present

    - name: Publish the welcome page
      ansible.builtin.copy:
        content: |
          Welcome to the web service
          Authorised access only
        dest: /var/www/html/index.html
        owner: root
        group: root
        mode: "0644"

    - name: Start and enable Apache
      ansible.builtin.systemd_service:
        name: httpd
        state: started
        enabled: true

    - name: Allow HTTP through firewalld
      ansible.builtin.include_role:
        name: redhat.rhel_system_roles.firewall
      vars:
        firewall:
          - service: http
            state: enabled
            runtime: true
            permanent: true

- name: Validate the web service from the control node
  hosts: localhost
  gather_facts: false
  become: false
  tasks:
    - name: Require an HTTP success response
      ansible.builtin.uri:
        url: "http://web1.example.com/"
        status_code: 200
```

The firewall task requires the `rhel-system-roles` package on the control node. The second play references `localhost`, so Ansible creates an implicit local host when inventory does not already define one. The URI check tests access from the control node. It does not confirm access from every client network.

`state: present` installs `httpd` without forcing an upgrade on every run. `state: latest` should appear only when the automation intentionally applies the newest available package. Ansible has no universal undo command. Administrators reverse a change by declaring the required replacement state, such as `state: absent`, or by restoring a tested backup or snapshot.

State-oriented modules usually support idempotent operation. After the first run establishes the requested state, another run should report `ok` instead of applying the same change again. Idempotence depends on each module and its arguments, so arbitrary command tasks require explicit safeguards.
## YAML essentials
YAML uses spaces to show hierarchy. Tabs cannot provide indentation. Two spaces per level form a common, readable convention, and sibling elements must align.

A mapping uses `key: value` pairs. Ansible accepts `key=value` as shorthand for arguments in some contexts, but that form is a module argument string rather than a YAML mapping. Separate YAML keys provide clearer types, diffs, and maintenance.

A sequence uses a hyphen for each item. A module accepts a list only when its documented argument type permits one. For example, `ansible.builtin.dnf` accepts several package names under `name`, while `ansible.builtin.systemd_service` accepts one unit name per task.

Plain, single-quoted, and double-quoted strings are available. Quoting values such as file modes avoids unintended type conversion. A literal block scalar introduced by `|` preserves line breaks. A folded block scalar introduced by `>` replaces most line breaks with spaces, which keeps a long logical line readable in the YAML source.
## Validation and execution
An explicit inventory makes each command's scope clear:

```shell
ansible-playbook -i inventory site.yml --list-hosts
ansible-playbook -i inventory site.yml --list-tasks
ansible-playbook -i inventory site.yml --syntax-check
ansible-playbook -i inventory site.yml --check --diff
ansible-playbook -i inventory site.yml
```

`--list-hosts` displays the hosts selected by every play, and `--list-tasks` displays tasks available before dynamic execution. These inspections help expose an incorrect host pattern or unexpected task import before execution.

`--syntax-check` parses the playbook without executing it. It can identify malformed YAML, invalid indentation, and some invalid Ansible structures, but it cannot detect a valid configuration that produces the wrong operational result.

`--check`, or `-C`, simulates supported tasks without changing managed hosts. Modules without check-mode support may skip work or return limited information. Tasks that depend on values registered by earlier changes can also behave differently during simulation. `--diff` displays supported before-and-after differences, although operators should protect output that may contain sensitive data. A check run therefore complements testing but does not replace it.

Normal execution reports each task and host as `ok`, `changed`, `failed`, `unreachable`, or `skipped`. `ok` means the task succeeded without changing state. `changed` means it succeeded and reported a change. The play recap also records rescued and ignored failures where applicable. Partial failure can leave hosts at different points, so operators must review the recap rather than relying on the final task alone.

A cautious rollout first targets a test inventory or a narrow `--limit`, confirms the resulting service, and then expands to the intended group. A successful command exit and a clean recap confirm Ansible's execution, while application checks confirm that the resulting system actually serves its purpose.

Repeated `-v` options increase diagnostic detail:

```shell
ansible-playbook -i inventory site.yml -vv
```

Moderate verbosity often reveals task arguments and results. Higher levels add connection and plugin details, increase output substantially, and can expose operational data. Operators should increase verbosity only as far as diagnosis requires.
## Managing multiple plays
Separate plays suit workflows that target different host groups or use different connection, user, privilege, or fact-gathering settings. A deployment can configure application servers, configure database servers, and then validate the service from the control node.

Each play may set its own `remote_user`, `become`, and `gather_facts` values. These settings support ordered orchestration across systems with different access requirements. They do not schedule plays independently, because one `ansible-playbook` run processes the plays as a single ordered workflow.

Large automation remains easier to test and maintain when roles, imported task files, or included task files hold reusable units. Each play should retain a clear purpose, and each task name should describe the state or action that its result represents.