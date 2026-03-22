# Getting Started with Playbooks
## Playbooks and plays
Ansible playbooks turn repeated command-line actions into reusable YAML files. A playbook is a list of plays. Each play selects target hosts and runs an ordered list of tasks to drive those hosts to a defined state. A play usually includes a descriptive `name`, a `hosts` selector, and `tasks`. The name is optional but strongly recommended because Ansible prints it during execution and troubleshooting.

A basic play can install a package and start its service in one pass.

```yaml
---
- name: Install and start httpd
  hosts: all
  tasks:
    - name: Install package
      yum:
        name: httpd
        state: installed
    - name: Start and enable service
      service:
        name: httpd
        state: started
        enabled: yes
```

This structure replaces separate ad hoc commands with a single, repeatable definition.
## YAML rules
YAML relies on indentation to show hierarchy. Each level must line up consistently, and spaces must be used instead of tabs. A playbook begins as a list, so each play starts with `- name:` beneath the optional document marker `---`.

Standard YAML uses `key: value` pairs. Some Ansible module arguments can be passed in compact forms in limited contexts, but readable playbooks work best when arguments are written as properly indented mappings on separate lines. That approach reduces syntax mistakes and makes complex tasks easier to review.

Lists hold multiple values under one key. Package installation is a common example.

```yaml
- name: Install packages
  yum:
    name:
      - nmap
      - httpd
      - vsftpd
    state: latest
```

Strings may appear without quotes unless quoting improves clarity. Multiline text uses two common styles. The `|` form preserves line breaks exactly. The `>` form folds wrapped lines into a single line. That distinction matters when playbooks create files with the `copy` module.
## Running playbooks
`ansible-playbook playbook.yml` runs the selected plays, gathers facts by default, executes each task, and ends with a play recap. The output shows the play name first, then each task name, then a status for each managed host.

The most useful status values are straightforward:
- `ok` means the task succeeded and the host already matched the desired state
- `changed` means the task succeeded and Ansible modified the host
- `failed` means the task stopped with an error
- `unreachable` means Ansible could not connect to the host

Fact gathering appears before user-defined tasks because Ansible collects system information that later tasks can use. The final recap summarises the result for every host.

Ansible does not provide a general undo feature for playbooks. Reversal requires another playbook that defines a new desired state, such as stopping a service, removing a package, or restoring a previous file.
## Syntax checks and dry runs
Syntax errors usually come from bad indentation or misplaced keys. `ansible-playbook --syntax-check playbook.yml` identifies parsing problems before a full run. A normal playbook run also stops on syntax errors, so syntax checking is mainly a quick validation step during editing.

`ansible-playbook -C playbook.yml` runs in check mode. It reports the changes Ansible would make without applying them. This is a practical way to estimate impact before touching managed nodes, although results depend on whether individual modules support check mode.

Verbose output helps when a playbook fails, but more detail is not always better. `-v` adds task results, `-vv` adds more task detail, `-vvv` adds connection information, and `-vvvv` exposes low-level debugging output. Moderate verbosity is usually enough. Excessive output hides the real problem under transport and plug-in noise.
## Multiplay playbooks
A single playbook can contain multiple plays. This is useful when one play configures remote systems and another validates the result from a different execution context. For example, one play can install and start `httpd` on managed hosts, while a second play runs on `localhost` and uses the `uri` module to test whether a web page is reachable and returns HTTP 200.

This pattern separates configuration from verification. It also shows that success inside a managed host does not guarantee end-to-end availability. A service may be installed and running while a firewall rule, routing problem, or other connectivity issue still blocks access from the control node.

Multiplay playbooks can also override connection behaviour per play, such as changing privilege escalation or the remote user. Even so, large files quickly become hard to reason about. Smaller, focused playbooks and reusable includes are usually easier to test, debug, and maintain.
## Practical guidance
A first practice playbook can install and enable `vsftpd`. Another useful exercise compares the `|` and `>` multiline string styles by writing two files and inspecting the results. A third exercise can build a simple web server playbook with a follow-up validation play on `localhost`.

The most reliable habit is to keep each play simple, name every play and task clearly, and let the structure explain the intent. Most playbook failures come from logic and layout mistakes rather than obscure Ansible internals. Clear indentation, small plays, and deliberate validation steps prevent most of those errors.