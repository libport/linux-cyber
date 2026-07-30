# Creating Shell Scripts
> [!NOTE]
> How to build reliable, secure, and maintainable Bash scripts on RHEL using command-line testing, validation, exit statuses, arguments, loops, functions, and safe input handling. 
## Shell scripts and automation
Bash serves as both a command interpreter and a programming language. A shell script stores commands, expansions, tests, loops, and functions in a text file so that Bash can execute them as a repeatable process. Scripts reduce manual work, apply the same procedure consistently, and make administrative operations easier to review.

An administrator can test most shell syntax interactively before placing it in a script. Multi-line input works at the command line because Bash displays a continuation prompt until it receives a complete construct. This loop processes three greetings:

```bash
for message in hi salut ciao
do
  printf '%s\n' "$message"
done
```

A `for` loop processes a list. A `while` loop repeats commands while its test or command returns success. Every loop must make progress towards termination. Otherwise, it can continue indefinitely.

Scripts can call shell builtins, functions, external programs, and other scripts. They can also redirect input and output, accept arguments, and make decisions from command results. Before an administrator automates a privileged operation, the administrator should test it with harmless data, check its failure paths, and confirm that repeated execution will not damage an existing configuration.
## A practical working environment
A disposable, non-production RHEL 8 virtual machine allows an administrator to snapshot the system, test privileged operations, and restore the earlier state. The host needs Bash, standard command-line utilities, an editor, and authorised `sudo` or root access. User-creation tests should use invented names that cannot collide with real identities.

The command line provides immediate feedback. `type` distinguishes keywords, builtins, functions, and external commands, while local manual pages describe the installed versions:

```bash
type for
type read
type useradd
man bash
man useradd
man passwd
```

An administrator should develop a privileged script in stages. A dry run can print planned actions without changing the system. Later stages can add one operation at a time, inspect its status, and verify its result. This sequence exposes errors before a loop amplifies them across many accounts.

A `while` loop often updates a counter:

```bash
count=3

while (( count > 0 ))
do
  printf '%s\n' "$count"
  (( count-- ))
done
```

Arithmetic syntax avoids the string-oriented `[` command for integer work. The decrement makes the condition false after three iterations. A resource-creation loop also needs a policy for existing resources and failed iterations.
## Exit status and command logic
Every command returns an exit status. Bash stores the status of the most recent foreground pipeline in `$?`. Status `0` denotes success. A status from `1` to `255` denotes a failure or another non-success result whose meaning depends on the command. Shells do not expose negative exit statuses. Values outside the supported range undergo conversion, so scripts should use documented values within the range.

```bash
ls /etc/hosts
printf 'Status: %s\n' "$?"

ls /etc/hostz
printf 'Status: %s\n' "$?"
```

The first `printf` reports the status from `ls`. Any intervening command would replace that value. A script should therefore test a command directly when possible:

```bash
if getent passwd bob >/dev/null
then
  printf 'The bob account exists.\n'
else
  printf 'The bob account does not exist.\n'
fi
```

The `&&` operator runs its right-hand command only when the left-hand command succeeds. The `||` operator runs its right-hand command only when the left-hand command fails. These operators suit short decisions:

```bash
mkdir sales && cd sales
```

```bash
cd marketing 2>/dev/null || {
  mkdir -p marketing &&
  cd marketing
}
```

The brace group runs in the current shell, so a successful `cd` changes the current working directory. The grouped commands also prevent a later `cd` from running when `mkdir` fails. Longer logic normally reads more clearly as an `if` statement.

Pipelines require care because Bash normally returns the status of the last command in the pipeline. A script that must detect failure anywhere in a pipeline can enable `pipefail`:

```bash
set -o pipefail
```

An unconditional newline runs the next command after the preceding command finishes, regardless of success. Short-circuit operators instead make the second command depend on the first result. Their left-to-right association can produce surprising behaviour when a line combines several operators. Grouping or a full conditional makes the intended relationship explicit.

Redirection can suppress expected output without hiding unrelated failures. `>/dev/null` discards standard output, while `2>/dev/null` discards standard error. A script should suppress a diagnostic only when it handles the corresponding failure. Otherwise, the diagnostic helps identify a fault.

Commands define their own non-zero statuses. A script should not assume that every failure returns `1`. It can preserve a command's status before printing another message:

```bash
some_command
status=$?

if (( status != 0 ))
then
  printf 'some_command failed with status %s\n' "$status" >&2
fi
```
## The interpreter directive
An executable Bash script normally begins with an interpreter directive:

```bash
#!/bin/bash
```

The kernel reads the `#!` characters at the start of an executable file and uses the remaining path as the interpreter. Bash treats that line as a comment after the kernel starts the interpreter. The directive must occupy the first line, and the interpreter path must exist on the target system. RHEL 8 provides Bash at `/bin/bash`.

The directive controls execution only when a process invokes the script as a program. The command `bash script.sh` starts Bash explicitly, so the named interpreter reads the file regardless of its execute permission or its first line. Direct execution requires execute permission:

```bash
chmod u+x script.sh
./script.sh
```

`chmod u+x` grants execution to the owner. `chmod a+x` grants it to the owner, group, and others. When the mode omits `u`, `g`, `o`, or `a`, the current `umask` can restrict which classes receive newly added permissions. An explicit class avoids that ambiguity.

On RHEL 8, `/bin/sh` commonly resolves to Bash operating with `sh`-compatible behaviour. It is not the historical Bourne shell. A Bash-specific script should name Bash in its interpreter directive rather than depend on whichever program provides `sh`.
## Script location and PATH
Bash searches the colon-separated directories in `PATH` when a command contains no slash. The current directory normally does not appear in `PATH`, so an executable in that directory requires a relative or absolute path such as `./script.sh`.

A personal directory such as `$HOME/bin` provides a suitable location for a user's scripts when the login configuration adds it to `PATH`. The home directory itself should not enter `PATH`. An administrator should inspect the active value rather than assume a distribution or account configuration:

```bash
printf '%s\n' "$PATH"
command -v script.sh
```

`command -v` reports how Bash would resolve a command. After an administrator creates `$HOME/bin` during a session, the administrator may need to update `PATH` or start a new login session before Bash can find scripts there.

An editor can reduce repeated typing. For example, a Vim insert-mode abbreviation in `$HOME/.vimrc` can expand a short token into the Bash interpreter directive:

```vim
iabbrev _sh #!/bin/bash
```

Vim reads `$HOME/.vimrc` when it starts, and the `vim-enhanced` package supplies the full Vim implementation on RHEL 8. The administrator still needs to confirm that the abbreviation expanded on line 1. `bash -n script.sh` checks syntax without executing commands, but tests must still cover success, failure, missing input, existing resources, and interrupted execution.
## Positional and special parameters
A script receives command-line arguments as positional parameters. Quoting preserves spaces, wildcard characters, and empty values.

| Parameter | Meaning |
| --- | --- |
| `$0` | The name used to invoke the shell or script |
| `$1`, `$2`, and so on | Individual positional parameters |
| `$#` | The number of positional parameters |
| `"$@"` | Every positional parameter as a separate quoted word |
| `"$*"` | All positional parameters joined into one quoted word |
| `$?` | The most recent foreground pipeline's exit status |
| `$$` | The process ID of the main shell |
| `$!` | The process ID of the most recent background job |

`$0` does not always contain an absolute path. It contains the invocation name supplied to Bash. Parameter expansion removes any leading path components without starting another program:

```bash
program_name=${0##*/}
```

`"$@"` is not a Bash array. It is a special expansion that produces one word for each positional parameter, which makes it the correct form for forwarding or iterating over arguments:

```bash
for user in "$@"
do
  printf '%s\n' "$user"
done
```

By contrast, `"$*"` joins all positional parameters into one word using the first character of `IFS`. Unquoted `$@` and `$*` allow word splitting and pathname expansion, so robust scripts rarely use those forms.

The interactive history designator `!$` recalls the last word of the preceding history entry. It does not represent a script parameter, and non-interactive shells normally disable history expansion. Bash uses `$!`, with the characters reversed, for the latest background process ID. Bash also updates `$_` to the final argument of the previous simple command, but scripts should store important values in named variables instead of relying on that changing value.

`$$` identifies the main Bash process. Inside some subshell contexts, `$$` retains the parent shell's value. Bash supplies `$BASHPID` when a script requires the process ID of the current Bash process.

Default-value expansion can supply a fallback for one parameter:

```bash
pid=${1:-1}
ps -f -p "$pid"
```

When a command must accept several process IDs, an array preserves each argument:

```bash
pids=("$@")

if (( ${#pids[@]} == 0 ))
then
  pids=(1)
fi

printf -v pid_list '%s,' "${pids[@]}"
pid_list=${pid_list%,}
ps -f -p "$pid_list"
```

Shell quoting controls when Bash treats text as one word. Unquoted variable expansion can split one value into several arguments and expand wildcard characters against filenames. Double quotes preserve parameter and command substitutions while preventing those two transformations. Single quotes preserve every enclosed character literally. A script should therefore quote expansions unless it deliberately requires splitting or pattern expansion.

Assignment syntax allows no spaces around `=`:

```bash
name=alice
```

`name = alice` asks Bash to run a command named `name`. Braces separate a variable name from adjacent text:

```bash
home_label="${name}_home"
```

Command substitution captures standard output:

```bash
program_name=$(basename -- "$0")
```

The shell removes trailing newlines from the captured output. Parameter expansion often avoids an external command, as `${0##*/}` does for a simple basename operation.

Arrays provide the safest representation for a list whose elements may contain spaces:

```bash
accounts=("alice smith" bob carol)

for account in "${accounts[@]}"
do
  printf '<%s>\n' "$account"
done
```

Linux login names normally cannot contain spaces, but the same quoting rule applies to paths, descriptions, and general script data. The distinction between a list and one joined string remains central to reliable Bash code.
## Tests and conditional statements
An `if` statement uses a command's exit status as its condition. It does not require `$?` or square brackets around a command:

```bash
if getent passwd "$user" >/dev/null
then
  printf 'Account already exists: %s\n' "$user"
fi
```

The `test` builtin and its `[` form evaluate file attributes, strings, and integers. Bash also provides `[[ ... ]]`, which avoids pathname expansion and supplies Bash-specific pattern and regular-expression matching. Arithmetic contexts use `(( ... ))`.

```bash
if (( $# < 1 ))
then
  printf 'Usage: %s USER [USER ...]\n' "${0##*/}" >&2
  exit 64
fi
```

`fi` closes an `if` construct. `elif` adds another condition without creating a separate nested statement. Diagnostic messages should go to standard error with `>&2`, and distinct exit statuses can help calling programs identify different failures.
## Reading input
The Bash `read` builtin collects input during execution. Its option names use lower case. `-p` displays a prompt, `-s` suppresses terminal echo, and `-r` prevents backslashes from acting as escape characters. If no variable name follows `read`, Bash stores the result in `REPLY`.

```bash
IFS= read -r -s -p 'Password: ' password
printf '\n'
```

Silent input prevents nearby observers from reading characters on the screen, but it does not validate, encrypt, or otherwise protect the stored value. A password should never appear as a command-line argument because process listings, shell history, logs, or audit systems can expose it.

A loop can reject an empty or whitespace-only value. The string test `-n` checks only whether a string has non-zero length, so spaces satisfy it. A regular expression can reject both cases:

```bash
while true
do
  if ! IFS= read -r -s -p 'Password: ' password
  then
    printf '\nPassword input ended.\n' >&2
    exit 1
  fi

  printf '\n'

  if [[ $password =~ [^[:space:]] ]]
  then
    break
  fi

  printf 'A password cannot be blank.\n' >&2
done
```

Password confirmation catches typing errors. System password policy still determines whether the password meets length, complexity, history, and dictionary requirements.

`read` returns a non-zero status when it reaches end of file or a timeout. A script that can receive redirected input should handle that result instead of looping forever. Interactive password prompts also require a terminal. Automation that runs without a terminal should obtain secrets through an approved secret-delivery mechanism, not through command-line arguments or source-code literals.

The `for` and `while` constructs solve different iteration problems. `for` suits a known argument list. `while` suits a condition that changes during execution. Both constructs return the status of the last command they execute, or `0` when they execute no commands. A script that needs an overall result from many iterations should track that result explicitly.

The `break` builtin leaves the nearest loop. `continue` skips the rest of the current iteration. These controls can simplify validation:

```bash
for user in "$@"
do
  if getent passwd "$user" >/dev/null
  then
    printf 'Skipped existing account: %s\n' "$user" >&2
    continue
  fi

  printf 'New account: %s\n' "$user"
done
```

This dry run performs no privileged change. It shows which arguments the script would process and which existing accounts it would skip.
## Functions
Functions give related commands a name and reduce duplication. Bash must read a function definition before the script calls it:

```bash
show_user()
{
  id "$1"
}

show_user alice
```

The empty parentheses form part of the function-definition syntax. They do not declare or receive parameters. A function call supplies arguments automatically through the function's own `$1`, `$2`, `$#`, and `"$@"`. A function can return an exit status from `0` to `255`, but it cannot return arbitrary text. It sends data through standard output or assigns an appropriately scoped variable.

Variables default to global scope in Bash functions. The `local` builtin prevents an internal variable from overwriting a variable elsewhere in the script:

```bash
account_exists()
{
  local user=$1
  getent passwd "$user" >/dev/null
}
```

A function should perform one coherent job, report diagnostics to standard error, and return a status that its caller can test. The calling code then controls whether a failure stops the script, skips one item, or records an overall failure and continues.

Function definitions improve readability only when their names describe actions accurately. Names such as `account_exists`, `read_password`, and `create_user` expose the main control flow. Excessively small functions can obscure simple operations, while large functions can conceal several unrelated responsibilities.

Functions execute in the current shell unless a surrounding construct creates a subshell. They can therefore change global variables, the working directory, shell options, and positional parameters. `local` variables and careful status handling limit unintended effects.
## User-account automation
Local account administration requires root privileges. `useradd --create-home NAME` creates an account and home directory. `getent passwd NAME` checks the configured name-service sources, not only `/etc/passwd`. On RHEL 8, `passwd --stdin NAME` can read a password from standard input. Other distributions may omit that option.

A multi-user script should process `"$@"`, quote every expansion, prompt separately for each account, and preserve a non-zero final status when any account fails. Reusing one password for several users weakens security and should not form part of an administrative workflow.

```bash
#!/bin/bash
usage()
{
  printf 'Usage: %s USER [USER ...]\n' "${0##*/}" >&2
}

read_password()
{
  local user=$1
  local first
  local second

  unset USER_PASSWORD

  while true
  do
    if ! IFS= read -r -s -p "Password for $user: " first
    then
      printf '\nPassword input ended.\n' >&2
      return 1
    fi

    printf '\n'

    if ! IFS= read -r -s -p 'Retype password: ' second
    then
      printf '\nPassword input ended.\n' >&2
      return 1
    fi

    printf '\n'

    if [[ -z $first || $first =~ ^[[:space:]]*$ ]]
    then
      printf 'A password cannot be blank.\n' >&2
    elif [[ $first != "$second" ]]
    then
      printf 'The passwords do not match.\n' >&2
    else
      USER_PASSWORD=$first
      return 0
    fi
  done
}

create_user()
{
  local user=$1

  if getent passwd "$user" >/dev/null
  then
    printf 'Account already exists: %s\n' "$user" >&2
    return 2
  fi

  if ! useradd --create-home "$user"
  then
    printf 'Account creation failed: %s\n' "$user" >&2
    return 3
  fi

  if ! read_password "$user"
  then
    printf 'Password input failed: %s\n' "$user" >&2
    return 4
  fi

  if ! printf '%s\n' "$USER_PASSWORD" |
    passwd --stdin "$user" >/dev/null
  then
    printf 'Password assignment failed: %s\n' "$user" >&2
    unset USER_PASSWORD
    return 5
  fi

  unset USER_PASSWORD
  id "$user"
}

if (( EUID != 0 ))
then
  printf 'The script requires root privileges.\n' >&2
  exit 77
fi

if (( $# == 0 ))
then
  usage
  exit 64
fi

overall_status=0

for user in "$@"
do
  if ! create_user "$user"
  then
    overall_status=1
  fi
done

exit "$overall_status"
```

The root check runs before any account operation. The argument check prevents an empty invocation from succeeding silently. The outer `for` loop preserves each argument through `"$@"`, calls `create_user` once per name, and records whether any call fails.

`create_user` queries the name-service database before it changes the system. This check can detect local and centrally managed identities. `useradd` still performs its own validation and remains the authority on whether it can create the requested local account. A race between the check and creation can still cause `useradd` to fail, so the script tests both operations.

`read_password` uses local variables for the two entries and assigns the confirmed value to a global variable only long enough for the caller to use it. `printf` sends the value through a pipe without placing it in the process command line. RHEL's `passwd --stdin` applies the configured password policy and stores the resulting password hash. The script immediately unsets the clear-text shell variable after the command returns.

Each function emits its own diagnostic and non-zero status. The main loop continues so one existing or invalid account does not prevent later arguments from receiving attention. The final status remains non-zero when any iteration fails, which allows a calling program to detect an incomplete batch.

The script leaves a newly created account locked if password assignment fails, which prevents password login but still requires an administrator to investigate and either complete or remove the account. Production automation should also define an approved username policy, log outcomes without secrets, handle interruption, and establish a deliberate rollback policy.

Account removal can delete valuable data. `userdel --remove NAME` removes the account, home directory, and mail spool, but it does not remove every file that the user owns elsewhere. An administrator should terminate active sessions, preserve required data, and confirm the exact account before removal.