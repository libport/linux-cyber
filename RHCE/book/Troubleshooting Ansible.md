# Troubleshooting Ansible
Ansible troubleshooting works best when execution stays safe, narrow, and observable. `ansible-playbook --syntax-check` validates playbook structure. `ansible-playbook --check` previews most changes without applying them. `--diff` shows file-level changes for supported modules, which is especially useful for templates. Check mode improves confidence, but it does not prove playbook logic. Tasks that depend on earlier changes may still fail during a real run.

Use task-level check mode to isolate risky changes:

```yaml
- name: Render Apache config safely
  ansible.builtin.template:
    src: httpd.conf.j2
    dest: /etc/httpd/conf/httpd.conf
  check_mode: true
```

`check_mode: false` forces a task to run even during a dry run, so it deserves caution.
## Using dry runs well
`--diff` matters most when templates or managed files change because it shows the exact lines that would move from the current state to the desired state. It is still not a complete test. Syntax checks catch YAML mistakes, dry runs preview supported changes, and real execution exposes missing directories, bad paths, failed handlers, and ordering mistakes. Each mode answers a different question, so reliable troubleshooting uses all three.
## Reading playbook output
Ansible output follows a predictable pattern:
- the play header
- fact gathering, unless disabled
- each named task
- the final recap

The recap reports whether each host stayed in the desired state, changed, failed, became unreachable, or skipped work. `rescued` shows that a block failed and a rescue section handled it. `ignored` shows that execution continued after an error because the playbook told Ansible to ignore it.

Verbosity controls depth:
- `-v` shows more result data
- `-vv` adds more request and response detail
- `-vvv` adds connection detail
- `-vvvv` exposes the full transport and privilege-escalation path

The highest level is useful for SSH and `become` faults, but it produces a large volume of output and may expose sensitive data. Log it to a file when deeper analysis is necessary.
## Making output easier to read
Only one stdout callback plugin can control console output at a time. A single callback such as `ansible.posix.debug` can make failures easier to scan. In current Ansible versions, the default callback can also format results more readably with YAML output.

```ini
[defaults]
log_path = /var/log/ansible.log
callback_result_format = yaml
callback_format_pretty = true
```

Persistent logs help with repeatable faults, but they grow quickly and need rotation.
## Running only the relevant tasks
Large playbooks are easier to debug when execution starts from the point of failure or pauses between tasks.

- `--list-tasks` shows the execution plan
- `--step` prompts before each task
- `--start-at-task "task name"` resumes from a named task

This approach depends on clear task names. Anonymous tasks slow diagnosis.
## Using troubleshooting modules
The `debug` module prints messages or variable values and helps confirm assumptions about facts, registered variables, and conditionals.

The `uri` module checks whether an HTTP endpoint returns the expected content or status. That makes it useful for web services and APIs.

```yaml
- name: Verify web page content
  hosts: localhost
  become: false
  tasks:
    - name: Request the site
      ansible.builtin.uri:
        url: http://ansible2.example.com
        return_content: true
      register: site
      failed_when: "'welcome' not in site.content"

    - name: Show response body
      ansible.builtin.debug:
        var: site.content
```

The `stat` module inspects file state. Combined with `fail`, it turns file drift into an explicit error.

```yaml
- name: Check file ownership
  hosts: all
  tasks:
    - name: Read file metadata
      ansible.builtin.stat:
        path: /tmp/statfile
      register: stat_out

    - name: Stop if ownership is wrong
      ansible.builtin.fail:
        msg: /tmp/statfile owner must be root
      when: stat_out.stat.pw_name != 'root'
```

The `assert` module validates assumptions early. Prompted values arrive as strings, so numeric checks should convert them before comparison. Conditional expressions in `that` should use raw Jinja expressions without `{{ }}`.

```yaml
- name: Validate requested file size
  hosts: localhost
  vars_prompt:
    - name: filesize
      prompt: specify a file size in megabytes
  tasks:
    - name: Enforce size limits
      ansible.builtin.assert:
        that:
          - filesize | int >= 1
          - filesize | int <= 100
        fail_msg: file size must be between 1 and 100 MB
        success_msg: file size is valid

    - name: Create the file
      ansible.builtin.command:
        cmd: dd if=/dev/zero of=/bigfile bs=1M count={{ filesize | int }}
```
## Using tags to narrow execution
Tags let a playbook run only the tasks that matter for the current test cycle.

```yaml
- name: Install packages
  ansible.builtin.yum:
    name:
      - httpd
      - vsftpd
    state: present
  tags:
    - install
```

`ansible-playbook --tags install playbook.yml` runs only tagged installation work. `--skip-tags` excludes selected areas. Special tags also matter:
- `always` runs unless explicitly skipped
- `never` runs only when explicitly requested
- `tagged` selects all tagged tasks
- `untagged` selects all untagged tasks
- `all` selects the normal execution set

Tags do apply to dynamic includes such as `include_tasks` and `include_role`, but the tag applies to the include statement itself, not automatically to every included task. To propagate tags into included content, use `apply` or switch to static imports.
## Diagnosing connectivity and privilege escalation
Ansible needs a working connection path to the managed host, a usable Python interpreter on the target, and correct privilege escalation where required. Inventory may also need `ansible_host` when name resolution points to the wrong address or multiple addresses.

`ansible all -m ping` checks Ansible connectivity and confirms that the remote host can run the Python-based ping module and return `pong`. It does not send ICMP echo requests. Depending on configuration, the command can also expose authentication and `become` problems.

Common causes of failure include:
- the wrong `remote_user`
- missing SSH keys or broken SSH reachability
- `become` not enabled when root access is required
- `become_user` or sudo rules set incorrectly
- a target host without a usable Python interpreter

Verbose output often reveals whether the fault sits in SSH transport, interpreter discovery, or privilege escalation.