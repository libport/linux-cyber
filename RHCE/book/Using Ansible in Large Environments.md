# Using Ansible in Large Environments
RHEL 10 supplies `ansible-core` 2.16 for managing RHEL 9 and RHEL 10 nodes. Large environments require disciplined inventory design, controlled concurrency, and modular playbooks. These controls improve performance without sacrificing service availability or operational clarity.
## Targeting inventory
Ansible patterns select inventory names and groups. The implicit `all` group contains every host, while `ungrouped` contains hosts outside explicit groups.

| Intent | Pattern |
| --- | --- |
| Select two groups | `webservers:dbservers` |
| Select their intersection | `webservers:&staging` |
| Exclude a group | `webservers:!retired` |
| Combine operations | `webservers:dbservers:&staging:!retired` |
| Match inventory names | `web*.example.com` |

Commas can replace colons as separators and work better with IPv6 addresses and ranges. Shell commands should quote patterns that contain wildcards, `!`, or `&`:

```shell
ansible 'webservers:&staging:!retired' -m ansible.builtin.ping
```

Ansible evaluates unions first, intersections second, and exclusions last. The `--limit` option intersects another pattern with the play's `hosts` selection, which supports a controlled subset without editing the playbook.

Patterns resolve inventory names, not arbitrary DNS results. If inventory defines `web1` with `ansible_host: 192.0.2.10`, the pattern must use `web1`. A trailing comma can create a temporary host-list inventory, as in `-i '192.0.2.10,'`, but that source provides none of the variables attached to the normal inventory.
## Dynamic and multiple inventory sources
Changing infrastructure should use an inventory plugin supplied by `ansible-core` or an installed collection. Administrators should prefer plugins to custom executable scripts because plugins integrate with current inventory processing, configuration, and caching. The old Python 2 script and `ec2.py` plus `ec2.ini` model should not guide RHEL 10 deployments. Legacy scripts still work through `ansible.builtin.script` when they return valid JSON and implement the required interface.

Plugin configuration normally uses YAML, a fully qualified plugin name, and the filename pattern required by that plugin. Administrators should obtain credentials from approved secret stores or Automation Platform credentials, restrict cache permissions, and set cache lifetimes that suit the rate of infrastructure change.

`ansible-inventory` reveals the compiled result:

```shell
ansible-inventory -i inventory --graph
ansible-inventory -i inventory --list
ansible-inventory -i inventory --host web1
```

Several `-i` options can combine static files, plugin configurations, directories, and scripts. An inventory directory aggregates supported sources and loads its top level alphabetically. Source order affects group construction and variable precedence. When several sources define the same variable, the last loaded value wins. Clear filenames, separate `group_vars` and `host_vars`, and review with `ansible-inventory` reduce accidental overrides.

Production, testing, and development should normally remain separate inventory sources. Administrators should combine them deliberately and constrain the selected hosts with an exact play pattern or `--limit`.
## Controlling execution
The default `linear` strategy runs one task across the current host batch before starting the next task. Ansible uses five forks by default.

| Control | Scope | Effect |
| --- | --- | --- |
| `forks` or `-f` | Controller or command | Sets the maximum worker count |
| `serial` | Play | Completes the play on one host batch before starting the next |
| `throttle` | Task or block | Lowers concurrency for a costly or rate-limited operation |
| `strategy: free` | Play | Lets each host advance independently |

Administrators should tune forks against controller CPU and memory, network capacity, connection overhead, managed-service load, and API limits. Raising the value to 100 solely because targets run Linux can overload the controller or the service being changed.

The `free` strategy improves throughput when hosts can progress independently, but it removes task-by-task lockstep between hosts. Coordinated cluster changes usually need the linear strategy, explicit batches, failure thresholds, and service health checks.

Rolling work uses `serial` in the play, not `ansible.cfg`:

```yaml
---
- name: Update the web tier in batches
  hosts: webservers
  serial: 3
  become: true
  tasks:
    - name: Apply the service update
      ansible.builtin.include_tasks: tasks/update_service.yml
```

Ansible completes the play for three hosts before taking the next batch. `serial` also limits the default failure scope to the active batch. A percentage or sequence of batch sizes can support staged rollouts. `throttle` can reduce workers below the `forks` or `serial` ceiling, but it cannot raise that ceiling.
## Reusing playbook content
Static imports expand during parsing. Dynamic includes load when execution reaches them.

| Statement | Behaviour and limits |
| --- | --- |
| `ansible.builtin.import_playbook` | Imports complete plays at the top level only |
| `ansible.builtin.import_tasks` | Imports a task list statically and exposes its tasks to listing and start-at-task operations |
| `ansible.builtin.include_tasks` | Loads tasks dynamically and supports run-time conditions, inventory variables, and loops |
| `ansible.builtin.include_vars` | Loads variable files dynamically |

An import statement cannot use a loop, although tasks inside an imported file can. Keywords on `import_tasks`, including conditions and tags, apply to every imported task. A templated import filename can use only values available during parsing, such as playbook variables or extra variables.

An include can respond to facts and earlier task results. Its internal tasks do not appear under `--list-tasks`, and `--start-at-task` cannot begin inside it. Tasks must notify a dynamic handler include as a unit. They can notify individual handlers from a static handler import.

Top-level orchestration can import specialised playbooks:

```yaml
---
- name: Load web server plays
  ansible.builtin.import_playbook: webservers.yml

- name: Load database plays
  ansible.builtin.import_playbook: databases.yml
```

Task files should accept variables instead of embedding environment-specific package, service, host, or port names. Roles suit larger reusable units that need defaults, handlers, files, templates, and metadata. Consistent use of fully qualified collection names prevents ambiguity.

Before deployment, teams should use syntax checks, inventory graphs, host and task listings, check mode, and diff mode where the selected modules support them. Representative staging runs should confirm idempotence, batch behaviour, failure handling, and acceptable load.