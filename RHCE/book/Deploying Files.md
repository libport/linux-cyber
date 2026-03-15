# Deploying Files
## Managing files with Ansible
Ansible manages files most effectively with purpose-built modules instead of ad hoc shell commands. Core modules cover inspection, attribute changes, content edits, transfer, synchronisation, access control, and template rendering. `stat` inspects file state. `file` creates, removes, and updates files, directories, links, ownership, and permissions. `copy`, `fetch`, and `synchronize` move data between the control node and managed nodes. `lineinfile`, `blockinfile`, and `replace` edit existing text files. `acl` manages filesystem access control lists. `template` renders configuration files from Jinja2 templates.
## Inspecting and correcting file state
`stat` collects facts about a path, including whether it exists, its type, mode, owner, checksum, and readability. Tasks can register that output and act only when the current state differs from the required state. This supports idempotent playbooks and avoids unnecessary changes.

```yaml
- stat:
    path: /tmp/statfile
  register: st

- file:
    path: /tmp/statfile
    mode: '0640'
  when: st.stat.mode != '0640'
```

`file` should also replace shell commands such as `touch` where possible. It expresses intent clearly and keeps the playbook idempotent.

The `file` module handles common filesystem tasks:
- create directories with `state: directory`
- create empty files with `state: touch`
- remove files or directory trees with `state: absent`
- set owner, group, mode, and links

A task name on every task makes failures easier to trace during troubleshooting.
## Editing file contents
`lineinfile` manages a single line that matches a regular expression. It suits settings such as `PermitRootLogin no` in `sshd_config`. `blockinfile` manages a multi-line block and wraps the managed text with begin and end markers, which makes later updates predictable.

```yaml
- lineinfile:
    path: /etc/ssh/sshd_config
    regexp: '^PermitRootLogin'
    line: 'PermitRootLogin no'
  notify: restart sshd
```

`blockinfile` treats a literal block introduced with `|` as multiple lines. A folded block introduced with `>` collapses line breaks into one logical line, so it is unsuitable when the destination file needs separate lines.

`copy` can also write fixed text to a file, but `lineinfile` and `blockinfile` give tighter control over where changes land in existing content. `replace` suits broader regex-based substitutions across a file when one managed line or one managed block is too narrow. `find` remains useful when a playbook must locate files by name, path, age, size, or other properties before acting on them.
## Creating, removing, and transferring files
File deployment usually follows three patterns.

- `copy` sends files from the control node to managed nodes
- `fetch` retrieves files from managed nodes to the control node
- `synchronize` performs rsync-style synchronisation for larger trees or incremental updates

`fetch` does not drop all retrieved files into one flat directory. It creates a host-specific path under the destination so files from different managed nodes do not overwrite each other. A fetch to `/tmp` can therefore produce paths such as `/tmp/ansible1/etc/motd`.

`synchronize` does more than update an existing file. It can create, update, and remove files according to rsync behaviour and configured options. It is generally the better choice for larger content sets, while `copy` is simpler for smaller, direct transfers.
## Managing SELinux correctly
Persistent SELinux changes belong in policy, not only on individual files. `sefcontext` writes file-context rules into SELinux policy. `restorecon` then applies those rules to the filesystem. Directly setting a context with the `file` module can work, but a later relabel may overwrite that manual label.

A typical workflow is:

```yaml
- yum:
    name: policycoreutils-python-utils
    state: present

- sefcontext:
    target: '/web(/.*)?'
    setype: httpd_sys_content_t
    state: present

- command: restorecon -Rv /web
```

`policycoreutils-python-utils` supplies the tools required for `sefcontext` and `restorecon` on the managed node.

Other SELinux tasks use dedicated modules. `selinux` sets the system state to enforcing, permissive, or disabled, usually with the targeted policy. `seboolean` enables or disables booleans such as `httpd_read_user_content` and can make those changes persistent.

A non-default Apache document root requires more than an HTTP configuration change. Apache must point to the new directory, SELinux must label that path correctly, and the service must still be tested afterwards. A successful playbook run does not prove that the application now serves content correctly.
## Generating configuration with Jinja2 templates
Templates provide a cleaner approach than repeated line edits when configuration files have structure, repeated patterns, or host-specific values. A Jinja2 template can contain plain text, comments, variables, expressions, loops, conditionals, and filters. The `template` module renders the source `.j2` file on each managed node.

```yaml
- template:
    src: vhost.conf.j2
    dest: /etc/httpd/conf.d/vhost.conf
    mode: '0644'
```

Variables interpolate values such as host facts.

```jinja2
ServerName {{ ansible_facts['fqdn'] }}
ErrorLog logs/{{ ansible_facts['hostname'] }}-error.log
```

Control structures generate dynamic content. A `for` loop can iterate through inventory groups to build host lists. An `if` block can switch content according to a variable such as the package name or operating system.

Filters transform output during rendering. Common examples include `to_json`, `to_yaml`, and `ipaddr`. The correct Jinja2 filter syntax uses a single pipe, for example `{{ myvar | to_yaml }}`, not a double pipe.

If the `ansible_managed` variable is configured, templates can stamp a managed-file comment at the top of rendered files. That warning helps reduce accidental manual edits.
## Operational practice
Reliable file automation depends on a small set of habits. Handlers should restart or reload services only when a notifying task reports change. That pattern reduces needless service disruption and keeps runs predictable.

- prefer Ansible modules over `command` or `shell` unless no module fits the task
- use conditionals and handlers so playbooks change only what needs changing
- name tasks clearly
- verify the end result with service checks, file inspection, or application tests

These habits keep playbooks readable, repeatable, and safer to run across many hosts.