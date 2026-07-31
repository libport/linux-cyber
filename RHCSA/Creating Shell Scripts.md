# Creating Shell Scripts
> [!NOTE]
> How to build reliable, secure, and maintainable Bash scripts on RHEL using command-line testing, validation, exit statuses, arguments, loops, functions, and safe input handling. 

Shell scripts turn repeated command-line work into reusable procedures. A script can create accounts, inspect system state, process files, and apply the same checks each time it runs. This consistency reduces manual effort, but automation also repeats mistakes quickly. Administrators should therefore develop scripts on a test system, validate every assumption, and check each command that can fail.

Bash uses the same language for interactive commands and scripts, including commands, variables, expansions, redirections, tests, loops, and functions. Interactive Bash adds facilities such as command history, job control, and line editing that a non-interactive script may not provide. A script should not depend on those interactive facilities.
## Script files, variables, and quoting
A shell script is a plain-text file. Any editor that preserves plain text can create one. Consistent indentation does not change Bash's grammar, but it reveals the boundaries of conditions, loops, and functions. Comments begin with `#` and continue to the end of the line. The shebang is the only `#` line that the operating system interprets before Bash starts.

Variable assignments contain no spaces around `=`:

```shell
greeting='Hello'
user_name='alice'
```

Spaces would make Bash treat the first word as a command name. Lowercase names suit variables that belong to one script. Uppercase names traditionally identify exported environment variables and shell variables with established meanings, such as `PATH` and `HOME`.

Parameter expansion retrieves a value:

```shell
printf '%s, %s\n' "$greeting" "$user_name"
```

Braces separate a variable name from adjacent text:

```shell
archive="${user_name}_files.tar"
```

Double quotes allow parameter expansion and command substitution while preserving the expanded value as one word. Single quotes preserve every enclosed character literally. Unquoted expansions can undergo word splitting and filename expansion, so a value containing spaces or wildcard characters may turn into several unintended arguments. Scripts should quote expansions unless they deliberately require splitting or globbing.

Command substitution captures a command's standard output:

```shell
program_name=$(basename -- "$0")
```

Bash removes trailing newlines from that captured output. It does not preserve a command's status in `$?` after another command runs, so scripts should test important commands directly. `printf` gives controlled formatting and handles arbitrary data more consistently than `echo`, whose option and escape handling varies across implementations.
## Commands, status values, and simple logic
Every Bash command returns an integer status from 0 to 255. Status 0 denotes success. A non-zero status denotes failure or another condition that the command documents. Negative exit statuses do not occur. Bash reserves or commonly uses some values for special failures, so a script should document any status values that it assigns.

`$?` expands to the status of the most recently completed foreground pipeline. Another command immediately replaces that value, so a script should test the command directly or save the status at once.

```shell
getent passwd alice >/dev/null
status=$?

if (( status == 0 ))
then
  printf 'Account alice exists.\n'
fi
```

The `&&` and `||` operators form conditional command lists:
- `first && second` runs `second` only when `first` succeeds.
- `first || second` runs `second` only when `first` fails.

They suit short operations whose relationship remains obvious.

```shell
mkdir -p "$HOME/work/sales" && cd "$HOME/work/sales"
```

Longer or consequential operations benefit from an `if` statement because it makes each branch explicit.

```shell
if getent passwd alice >/dev/null
then
  printf 'Account alice already exists.\n' >&2
else
  sudo useradd -m alice
fi
```

A shell interprets a newline as a command separator. It also accepts a semicolon for that purpose, but multiline scripts normally use newlines. Redirection can suppress unwanted output, as in `>/dev/null`, or errors, as in `2>/dev/null`. A script should retain useful diagnostics instead of hiding failures indiscriminately.

A pipeline normally reports the status of its final command. That rule can conceal an earlier failure:

```shell
generate_report | compress_report
```

With `set -o pipefail`, the pipeline reports failure when any component fails. This option supports reliable automation, especially when the last command could succeed after receiving incomplete input. Each command can still define several non-zero results, so scripts should consult command documentation when they must distinguish absence, invalid input, permission failure, and system failure.

`&&` and `||` have equal precedence in an AND-OR list and associate from left to right. Mixing them without grouping can obscure intent. An `if` statement makes a multi-step recovery path clearer and prevents a later editor from misreading which failure triggers which action.
## Loops at the command line and in scripts
A `for` loop processes a list. The shell assigns each item to a variable, runs the body, and continues until it exhausts the list.

```shell
for message in hi salut ciao
do
  printf '%s\n' "$message"
done
```

A `while` loop repeats while its condition succeeds. The loop must normally change the state tested by that condition.

```shell
i=3

while (( i > 0 ))
do
  printf '%d\n' "$i"
  (( i-- ))
done
```

Both forms work interactively and in script files. At an interactive continuation prompt, Bash waits for the closing `done`. In a compact one-line command, separators must distinguish the loop header, body, and terminator.

A loop over `"$@"` preserves every argument exactly, including an argument that contains spaces. A loop over unquoted command substitution does not provide the same protection because Bash splits the resulting text and expands matching pathnames. Scripts should process structured input through positional parameters, arrays, or a line-reading loop instead of parsing `ls` output.

```shell
while IFS= read -r line
do
  printf 'Input: %s\n' "$line"
done < names.txt
```

`IFS=` prevents `read` from trimming leading or trailing whitespace, and `-r` prevents backslashes from escaping the next character. The loop stops when `read` reaches end of file. If the final line lacks a newline, a more elaborate condition may need to process the remaining text.

Loop bodies should quote data, check consequential commands, and guarantee progress. A `while` condition that never changes creates an infinite loop unless the body deliberately uses `break`, receives a signal, or terminates the script.
## Interpreters, permissions, and command lookup
A directly executed Bash script should begin with this shebang on RHEL 8:

```shell
#!/bin/bash
```

The kernel recognises `#!` only at the start of an executable file and launches the named interpreter. Bash treats the same line as a comment after the interpreter starts. The shebang therefore controls direct execution:

```shell
chmod u+x report.sh
./report.sh
```

`chmod u+x` grants execute permission to the file owner. An omitted class in `chmod +x` allows the current `umask` to influence which execute bits change. `chmod a+x` grants execute permission to the owner, group, and others, which often exceeds what a personal script needs.

Running `bash report.sh` does not require execute permission because Bash opens the file as input. That form also selects Bash explicitly and does not rely on the file's shebang. Direct execution uses the shebang and requires execute permission.

Running `sh report.sh` selects the system's `sh` implementation, not necessarily Bash with its normal behaviour. On RHEL 8, `sh` invokes Bash in a compatibility mode, which can change or reject Bash-specific syntax. A script that uses `[[ ... ]]`, arrays, or other Bash features should declare and use Bash.

Sourcing a file with `source report.sh` or `. report.sh` differs from executing it. Sourcing runs commands in the current shell, so directory changes, variable assignments, options, and function definitions can persist afterwards. Normal execution isolates most such changes in the script process. Administrative scripts should use normal execution unless they intentionally modify the caller's environment.

The shell searches the directories in `PATH` when a command contains no slash. It does not normally search the current directory. `./report.sh` supplies an explicit relative path and therefore bypasses command lookup.

Personal scripts can reside in `$HOME/bin` when the user's startup configuration includes that directory in `PATH`.

```shell
mkdir -p "$HOME/bin"
mv report.sh "$HOME/bin/"
export PATH="$HOME/bin:$PATH"
command -v report.sh
```

The `export` command above affects the current shell and its descendants. A suitable assignment in `~/.bash_profile` or another applicable startup file makes the change available after later logins. RHEL installations and user profiles can differ, so each user should inspect the actual `PATH` instead of assuming that `$HOME/bin` already appears there. `command -v` reports how Bash would resolve a command and avoids relying on the less consistent external `which` utility.
## Positional and special parameters
Arguments supplied after a script name become positional parameters. Bash temporarily replaces a function's positional parameters with the arguments supplied to that function.

| Expansion | Meaning |
| --- | --- |
| `$0` | The name or path used to invoke the script. It does not always contain an absolute path. |
| `$1` to `$9` | The first nine positional parameters. |
| `${10}` | The tenth positional parameter. Braces prevent Bash from reading it as `$1` followed by `0`. |
| `$#` | The number of positional parameters. |
| `"$@"` | Every positional parameter as a separate quoted word. |
| `"$*"` | All positional parameters as one quoted word, joined by the first character of `IFS`. |
| `$?` | The status of the most recent foreground pipeline. |
| `$$` | The process ID associated with the shell. |
| `$!` | The process ID of the most recent background pipeline. |

`"$@"` preserves argument boundaries and therefore suits most loops:

```shell
for user in "$@"
do
  printf 'User: %s\n' "$user"
done
```

`"$*"` combines all arguments into one word. Unquoted `$@` and `$*` expose their results to word splitting and filename expansion, which can corrupt spaces, wildcard characters, and empty arguments. Scripts should normally quote parameter expansions.

An array and the special parameter `@` are related but not identical. Bash maintains positional parameters separately from named arrays. `"$@"` behaves usefully like a sequence of individually quoted words when Bash expands it in a context that performs word splitting. Calling it an array can hide this distinction and lead to invalid array syntax.

`!$` is not a special parameter. In an interactive shell with history expansion enabled, it expands to the final word of the previous history entry. Non-interactive scripts do not normally enable history expansion, so scripts should pass or store values explicitly. `$!`, with the characters reversed, has the unrelated background-process meaning shown above.

`$$` identifies the main shell process associated with the executing Bash instance. `BASHPID` identifies the current Bash process and can differ from `$$` in some subshell contexts. Most scripts need neither value, but process-management scripts should choose the one that matches their purpose.

`$0` preserves the form used at invocation. A script can obtain only its final path component when producing a usage message:

```shell
program_name=$(basename -- "$0")
printf 'Usage: %s USER...\n' "$program_name" >&2
```
## Conditions and input
An `if` statement runs a command or test and chooses a branch from its status. Bash closes the construct with `fi`.

```shell
if (( $# == 0 ))
then
  printf 'At least one user name is required.\n' >&2
  exit 1
fi
```

`test`, `[ ... ]`, and Bash's `[[ ... ]]` construct evaluate expressions. `[[ ... ]]` provides safer Bash-specific string handling and pattern matching. A command can serve as the condition without brackets:

```shell
if getent passwd "$user" >/dev/null
then
  printf 'Account %s already exists.\n' "$user" >&2
fi
```

`getent passwd "$user"` queries the configured name-service sources, not only `/etc/passwd`. This broader check helps prevent a local account from colliding with an account supplied by another configured identity source.

Arithmetic conditions use numeric comparisons, while string conditions compare text. Mixing them can produce incorrect branches.

```shell
if (( count > 0 ))
then
  printf '%d accounts remain.\n' "$count"
fi

if [[ $role == administrator ]]
then
  printf 'Privileged role selected.\n'
fi
```

File tests such as `[[ -e $path ]]`, `[[ -d $path ]]`, and `[[ -x $path ]]` examine existence, directory type, and execute permission. A successful pre-check cannot guarantee that a later operation will succeed because permissions or system state can change between the check and the action. Scripts must still test the action itself.

The `read` builtin gathers input during execution. `-r` preserves backslashes, `-s` suppresses terminal echo, and `-p` displays a prompt. Silent input reduces shoulder-surfing risk, but it does not encrypt the value or remove it from the script's memory. A password should not appear as a command-line argument because process listings, logs, or shell history may expose it.

```shell
IFS= read -r -s -p "Password for $user: " password
printf '\n'
```

An empty string and a whitespace-only string require different checks. `[[ -n $password ]]` accepts spaces because spaces give the string a non-zero length. A loop can require at least one non-whitespace character:

```shell
while true
do
  IFS= read -r -s -p "Password for $user: " password
  printf '\n'

  if [[ $password =~ [^[:space:]] ]]
  then
    break
  fi

  printf 'Password cannot be empty or whitespace only.\n' >&2
done
```
## Functions and structured automation
A function gives a related command sequence a name. Bash executes a function in the current shell context, so variable changes can affect the surrounding script unless the function declares local variables.

```shell
show_user() {
  local user=$1
  getent passwd "$user"
}

show_user alice
```

The empty parentheses belong to the function-definition syntax. They do not declare or receive named parameters. Arguments supplied in the function call become `$1`, `$2`, and the function's other positional parameters. Functions should use `return` to report a status to their caller. `exit` terminates the entire script.

A function's variables remain global unless `local` limits their scope. Local names prevent one function from overwriting state used elsewhere. A function can return only an integer status. To produce text or records, it can write to standard output and let the caller capture that output through command substitution. Diagnostic messages belong on standard error so data pipelines do not consume them accidentally.

Function definitions must execute before Bash reaches the corresponding calls. Scripts commonly place definitions near the top and keep a short main routine at the bottom. This organisation separates reusable operations from execution order.

Clear functions separate validation, account creation, password handling, and reporting. The main loop can then show the overall control flow without duplicating commands.
## Account creation with validation
Account creation requires administrative authority and careful password handling. Each new account should receive its own password. Reusing one password across several accounts creates an avoidable security weakness.

The script below accepts one or more user names, refuses existing accounts, prompts twice for each new password, creates a home directory, and reports failures. RHEL 8 supplies the `passwd --stdin` option used here. Other distributions may require a different password-setting command.

```shell
#!/bin/bash
set -o nounset
set -o pipefail

usage() {
  printf 'Usage: %s USER...\n' "$(basename -- "$0")" >&2
}

set_password() {
  local user=$1
  local password=
  local confirmation=

  while true
  do
    IFS= read -r -s -p "New password for $user: " password
    printf '\n'

    if [[ ! $password =~ [^[:space:]] ]]
    then
      printf 'Password cannot be empty or whitespace only.\n' >&2
      continue
    fi

    IFS= read -r -s -p "Repeat password for $user: " confirmation
    printf '\n'

    if [[ $password == "$confirmation" ]]
    then
      break
    fi

    printf 'Passwords do not match.\n' >&2
  done

  if printf '%s\n' "$password" |
    sudo passwd --stdin "$user" >/dev/null
  then
    unset password confirmation
    return 0
  fi

  unset password confirmation
  printf 'Password setup failed for %s.\n' "$user" >&2
  return 1
}

create_user() {
  local user=$1

  if getent passwd "$user" >/dev/null
  then
    printf 'Skipping %s because the account already exists.\n' "$user" >&2
    return 1
  fi

  if ! sudo useradd -m -- "$user"
  then
    printf 'Account creation failed for %s.\n' "$user" >&2
    return 1
  fi

  if ! set_password "$user"
  then
    printf 'The new account %s remains locked until an administrator sets a password.\n' "$user" >&2
    return 1
  fi

  getent passwd "$user"
}

if (( $# == 0 ))
then
  usage
  exit 1
fi

if ! sudo -v
then
  printf 'Administrative authorisation failed.\n' >&2
  exit 1
fi

status=0

for user in "$@"
do
  if ! create_user "$user"
  then
    status=1
  fi
done

exit "$status"
```

`sudo -v` validates administrative access before password collection begins. `useradd -m` creates the account and its home directory. A newly created account remains locked if password setup fails, and the diagnostic states that condition. The script clears its password variables after use, although the shell cannot guarantee complete erasure from process memory.

The pipeline sends the password through standard input rather than placing it in the command's argument list. `set -o pipefail` makes the pipeline fail when either `printf` or `passwd` fails. Each function returns a status, and the loop records whether any account failed.

`getent` performs the collision check before `useradd`, but `useradd` remains the authoritative creation attempt. Another process could create the same account between those operations. The explicit `useradd` check catches that race and prevents password handling from continuing after a failed creation.

The script does not delete a newly created account automatically when password setup fails. Automatic cleanup could remove files or state created concurrently by another process. Instead, the locked account and clear diagnostic leave an administrator to inspect the result and choose recovery or deletion. Production automation can implement a stronger transaction policy when it controls all related state.
## Reliable development
Small, testable changes make shell automation safer. Administrators should first run commands manually on a disposable RHEL 8 system, then place the working sequence in a script and add validation around it. Useful checks include:
- `bash -n script.sh` parses Bash syntax without executing the script.
- `bash -x script.sh` traces expanded commands during execution.
- `command -v name` verifies how Bash resolves a command.
- Explicit status checks confirm that privileged or destructive commands succeeded.

Tracing can expose expanded secrets. A script should disable tracing around password handling and should never enable it for the account-creation example without first removing the password data from traced commands.

Comments should explain decisions, constraints, and unusual behaviour. Names such as `user`, `password`, and `create_user` communicate intent more effectively than unexplained single-letter names. Quoted expansions, local function variables, distinct error messages, and staged testing make failures easier to diagnose.

Automation should stop or report clearly when its assumptions fail. Scripts that create, change, or delete accounts need stricter checks than scripts that print information. Successful execution alone does not prove that the resulting system state is correct, so verification should inspect the account, home directory, permissions, and intended login behaviour.

Repeatable scripts aim for idempotent outcomes where practical. Running an account-creation script again should detect an existing account and avoid replacing its password or ownership without explicit authority. Least-privilege permissions, narrow `sudo` access, validated input, and visible failures reduce the consequences of an error. A separate test environment remains essential because even well-checked commands can alter system state.