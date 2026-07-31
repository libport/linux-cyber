# Deploying Files
Ansible manages file attributes, content, transfers, SELinux labels, and generated configuration. RHEL 10 control nodes include `ansible-core` 2.16 for RHEL System Roles. Fully qualified collection names identify each module and expose dependencies outside `ansible-core`.
## Selecting a file module
Each operation should use the narrowest suitable module.

| Requirement | Module |
|---|---|
| Inspect a path | `ansible.builtin.stat` |
| Manage directories, links, ownership, modes, or deletion | `ansible.builtin.file` |
| Deploy static content from the control node | `ansible.builtin.copy` |
| Manage one line | `ansible.builtin.lineinfile` |
| Manage a marked text block | `ansible.builtin.blockinfile` |
| Replace every matching expression | `ansible.builtin.replace` |
| Render variable data into a file | `ansible.builtin.template` |
| Retrieve a remote file | `ansible.builtin.fetch` |
| Locate remote paths | `ansible.builtin.find` |
| Synchronise directory trees with rsync | `ansible.posix.synchronize` |

`stat` returns data under a registered variable's `stat` key. A permission test therefore reads a value such as `result.stat.mode`, which is a string such as `'0640'`. A task that only needs to enforce attributes can call `file` directly because the module already reports `ok` when the path complies. `stat` remains useful for auditing or branching on existence, type, checksum, ownership, or timestamps. SHA-256 avoids the MD5 restriction on FIPS-enabled systems.

File modes should always use quoted octal strings such as `'0640'` or symbolic forms such as `u=rw,g=r,o=`. `state: file` changes an existing file but does not create one. `state: touch` creates a missing file and updates timestamps on every run unless `access_time` and `modification_time` use `preserve`. `state: absent` removes a directory and its contents recursively.

Direct state enforcement usually needs no preliminary test:

```yaml
- name: Create the application directory
  ansible.builtin.file:
    path: /srv/example
    state: directory
    owner: root
    group: apache
    mode: '0750'

- name: Create a marker without changing existing timestamps
  ansible.builtin.file:
    path: /srv/example/.managed
    state: touch
    owner: root
    group: apache
    mode: '0640'
    access_time: preserve
    modification_time: preserve
```

`lineinfile` manages one line. Its regular expression should match both the original line and the replacement so repeated runs stay idempotent. `blockinfile` maintains several lines between marker lines, and each managed block in one file needs a distinct marker. `replace` suits repeated regular-expression substitutions. A template is safer when automation owns most of a configuration file. The `validate` option can test a temporary candidate before `copy`, `lineinfile`, or `template` replaces the live file.

Configuration tasks should set ownership and mode explicitly. `backup: true` preserves the previous destination when a module changes it. `ansible-playbook --check --diff` can preview supported changes, although a syntax check or check-mode run cannot prove that the resulting service will work. Diffs can expose secrets, so sensitive tasks need appropriate output controls.
## Transferring files
`copy` transfers a file only when its content or managed metadata differs. It does not create a fresh file on every run. `ansible.posix.synchronize` wraps rsync for efficient directory-tree transfers, originates on the control or delegated host, and requires rsync on both ends. It belongs to the `ansible.posix` collection rather than `ansible-core`.

By default, `copy.src` refers to the control node. `remote_src: true` instead reads an existing path on the managed host. The `content` parameter suits short, fixed text. A separate template should render structured content or variables. Large trees with hundreds of files favour `synchronize` because recursive `copy` does not scale well.

`fetch` copies in the opposite direction. With `src: /etc/motd` and `dest: /backup`, the default destination is `/backup/<inventory-host>/etc/motd`. `flat: true` removes that host-and-path hierarchy, but hosts with the same source basename can overwrite one another.
## Managing SELinux labels
RHEL 10 normally runs the targeted SELinux policy in enforcing mode. A non-standard service directory needs a persistent file-context mapping and labels applied from that policy. Setting `setype` directly through `file`, `copy`, or `template` changes the current inode label but does not add a policy mapping, so a later relabel can undo it.

The `community.general.sefcontext` module manages persistent mappings and requires its collection plus the RHEL `policycoreutils-python-utils` package. It does not relabel existing files. `restorecon` must apply the mapping after the files exist.

```yaml
- name: Define the web content label
  community.general.sefcontext:
    target: '/web(/.*)?'
    setype: httpd_sys_content_t
    state: present

- name: Apply SELinux labels
  ansible.builtin.command: restorecon -irv /web
  register: restorecon_result
  changed_when: restorecon_result.stdout | length > 0
```

The relabelling task should not depend only on a handler notified by the mapping task. Existing mappings can remain unchanged while newly created or incorrectly labelled files still require `restorecon`.

The supported `redhat.rhel_system_roles.selinux` role can manage policy changes and restore directory contexts through `selinux_restore_dirs`. SELinux booleans should enable only policy-defined behaviour that the service requires. Broad permissions, writable content types, and permissive mode should not replace a specific policy correction.

Type selection follows the required access. Apache can read `httpd_sys_content_t` content, while content that the service must modify generally needs `httpd_sys_rw_content_t`. Filesystem ownership and mode still apply alongside SELinux. An administrator should verify both controls and inspect audit messages before changing policy.
## Generating files with Jinja
The `template` module renders UTF-8 Jinja source on the control node, then transfers the result to each managed host. Jinja uses three principal delimiters:

| Form | Purpose |
|---|---|
| `{{ expression }}` | Insert or transform a value |
| `{% statement %}` | Control flow with `for`, `if`, and related statements |
| `{# comment #}` | Add a template-only comment |

A loop can generate an inventory-based hosts file:

```jinja2
# Managed by Ansible
{% for host in groups['all'] %}
{% if hostvars[host]['ansible_facts']['default_ipv4'] is defined %}
{{ hostvars[host]['ansible_facts']['default_ipv4']['address'] }} {{ hostvars[host]['ansible_facts']['fqdn'] }} {{ host }}
{% endif %}
{% endfor %}
```

`groups['all']` supplies inventory names. `hostvars` exposes another host's variables, but its facts exist only after Ansible has gathered or cached them. Access through `hostvars[host]['ansible_facts']` does not depend on legacy fact injection.

Filters transform values inside expressions. `{{ data | to_json }}` produces JSON, and `{{ data | to_yaml }}` produces YAML. The source form with `|| to_yaml` is invalid. IP address processing uses the fully qualified `ansible.utils.ipaddr` filter, which requires the `ansible.utils` collection and the `netaddr` Python library on the control node.

Undefined data should fail clearly or receive an intentional default. Expressions can use `default()` for an approved fallback and `mandatory` when omission must stop rendering. Template source belongs in version control, and a managed-file comment should warn administrators that automation owns the destination. In `ansible-core` 2.16, `ansible_managed` can supply a configurable marker.

Templates that manage service configuration should notify a handler instead of restarting the service unconditionally:

```yaml
- name: Configure the Apache HTTP Server
  hosts: web
  become: true
  tasks:
    - name: Deploy the Apache configuration
      ansible.builtin.template:
        src: templates/httpd.conf.j2
        dest: /etc/httpd/conf.d/site.conf
        owner: root
        group: root
        mode: '0644'
        backup: true
      notify: Restart httpd

  handlers:
    - name: Restart httpd
      ansible.builtin.systemd_service:
        name: httpd
        state: restarted
```

RHEL 10 supplies Apache HTTP Server 2.4, so templates must use current directives such as `Require all granted` instead of the obsolete `Order` and `Allow` access controls. A successful playbook run confirms task execution, not service correctness. Syntax checks, service status checks, and functional requests should verify the deployed result.