# Creating Shell Scripts
## Core Bash principles
Shell scripting builds on command-line behaviour rather than replacing it. If a command or control structure works in the shell, the same logic can usually move into a script with only minor formatting changes. This makes the command line the fastest place to test ideas before they become part of a reusable tool.
### Using loop syntax at the command line
A `for` loop processes each item in a list. At the prompt it can print greetings, create several directories or add several users with similar settings. In a script the same structure becomes easier to read because each keyword can sit on its own line.

```bash
for msg in hi salut ciao
do
  echo "$msg"
done
```

A `while` loop repeats while a test remains true. It suits counters, retries and validation.

```bash
i=3
while [ "$i" -gt 0 ]
do
  echo "$i"
  i=$((i - 1))
done
```

These patterns matter because shell scripts rarely do anything magical. They organise familiar commands into clear, repeatable flows. A loop that works interactively will usually work unchanged inside a script once it is placed in a file and given the right interpreter.
### Exit status and simple logic
Bash relies on exit status to decide whether a command succeeded. An exit status of `0` means success. Any non-zero status indicates failure. In practice Bash exposes statuses in the range `0` to `255`, so negative values do not appear as shell exit codes.

The shell stores the most recent exit status in `$?`. Scripts can inspect that value directly, but Bash also provides compact control operators that make simple logic easier to read.

- `cmd1 && cmd2` runs `cmd2` only if `cmd1` succeeds
- `cmd1 || cmd2` runs `cmd2` only if `cmd1` fails

This pattern is useful for directory creation and navigation:

```bash
mkdir sales && cd sales
```

It also helps when a directory might already exist.

```bash
cd mkt 2>/dev/null || {
  mkdir mkt &&
  cd mkt
}
```

The grouped command on the right keeps directory creation and the follow-up `cd` together so Bash evaluates them as one fallback path. Redirecting the failed `cd` message to `/dev/null` keeps the output tidy when the script already expects that failure.
### Command-line testing as part of script design
Interactive testing remains part of the design process. Administrators can confirm that shell keywords are real language elements with tools such as `type for`, inspect exit codes immediately after a command runs and refine logic before any script file exists. This habit matters in exams and in production because it shortens the path between an idea and a working result.

A practical example is bulk user creation. Instead of typing several `useradd` commands by hand, a short loop can create multiple accounts and then remove them again during testing. The same logic can later move into a script file with clearer spacing, error handling and prompts.
## Creating executable shell scripts
A reusable script needs the right interpreter, sensible permissions and a location that Bash can find.
### The shebang
The first line of a shell script should declare the interpreter.

```bash
#!/bin/bash
```

That line is the shebang. It tells the system to run the script with Bash even if the current shell is different. Without it, a script may execute in the caller's current shell, which can break Bash-specific syntax.

The point becomes obvious when a user launches a script from `sh` or another shell. Without a shebang, the script inherits that environment. With a shebang, the system starts Bash explicitly and the script behaves as designed. The same pattern selects Python, Perl or another interpreter when required.
### Execute permissions and PATH
A script can run without execute permission when Bash opens it explicitly.

```bash
bash my.sh
```

Direct execution requires the execute bit.

```bash
chmod a+x my.sh
./my.sh
```

Using `a+x` clearly applies execute permission to all classes. A plain `+x` depends on the current `umask`, so it may not grant execute permission to every class as intended.

Bash only finds commands in directories listed in `PATH`. On many RHEL systems `~/bin` is included when that directory exists, which makes it a sensible place for personal scripts.

```bash
mkdir -p ~/bin
mv my.sh ~/bin/
my.sh
```

`which my.sh` confirms which file Bash will run. When the full path is supplied, `PATH` is ignored and Bash runs exactly the named file. That distinction explains why `./my.sh` works in the current directory even when `my.sh` alone does not.
### Faster editing with Vim abbreviations
Vim can insert common boilerplate through abbreviations stored in `.vimrc`. An abbreviation for the shebang reduces repeated typing and keeps new scripts consistent.

```vim
abbr _sh #!/bin/bash
```

Typing `_sh` in insert mode expands to the full shebang. The feature is optional, but it speeds up routine editing. The same approach can store other frequent snippets, including interpreter lines for Python or Perl.

The editing workflow also highlights a broader point. Useful editor habits save time, but the script still needs clean structure, deliberate spacing and clear naming. Automation starts in the file itself, not only in the commands the file later runs.
## Special variables and argument handling
Bash provides built-in variables that describe the running script and the arguments passed to it.

- `$$` is the process ID of the current shell or script
- `$?` is the exit status of the previous command
- `$0` is the script name or path used to invoke it
- `$1`, `$2` and later values are positional parameters
- `$#` is the number of positional parameters
- `"$*"` expands all positional parameters as one word
- `"$@"` expands all positional parameters as separate words

These values make a script flexible. A process-inspection script, for example, can use a supplied process ID instead of relying on a hard-coded value. If no argument is provided, parameter expansion can supply a default.

```bash
PID="${*:-1}"
ps -f -p "$PID"
```

That example defaults to process `1` when no argument is supplied. It is more resilient than assuming the user always provides input.

`$0` is especially useful in usage messages because the script can name itself accurately.

```bash
echo "Usage: $0 username"
```

Running `basename "$0"` strips the path and prints only the file name. This is helpful when a script lives inside a long path but the message only needs the visible command name.

The last argument from the previous command can also be recalled interactively with `!$`. That shortcut belongs to shell history rather than to a script file, but it reinforces the same lesson. Bash stores useful context, and efficient scripting often begins with understanding what the shell already provides.
## Conditional structure and validation
Reliable scripts validate inputs before they make changes. Bash offers several direct ways to express that logic.
### Testing values
Square brackets and the `test` command both evaluate conditions. They are commonly used for argument counts, numeric comparisons and string checks.

```bash
if [ "$#" -lt 1 ]
then
  echo "Usage: $0 username"
  exit 1
fi
```

This pattern rejects an empty argument list and returns a deliberate exit status.
### Testing commands directly
When the decision depends on a command succeeding or failing, Bash does not need square brackets. It can use the command's exit status directly.

```bash
if getent passwd "$1" >/dev/null
then
  echo "User $1 already exists"
  exit 2
fi
```

This form is cleaner than running a command, reading `$?` separately and then writing another comparison. Bash already knows whether the command worked.
### Using `elif` to reduce clutter
Related checks can sit inside one `if` block instead of several disconnected tests.

```bash
if [ "$#" -lt 1 ]
then
  echo "Usage: $0 username"
  exit 1
elif getent passwd "$1" >/dev/null
then
  echo "User $1 already exists"
  exit 2
fi
```

`if` begins the conditional and `fi` closes it. Using `elif` keeps the decision tree compact and easier to follow.
## Building a user creation script
The central example is a script that creates local user accounts and sets passwords. The final version evolves in stages, which reflects good practice. Small increments make debugging easier and keep each change testable.
### Validate required input
A user creation script needs at least one user name. The first check should reject an empty argument list.

```bash
if [ "$#" -lt 1 ]
then
  echo "Usage: $0 username"
  exit 1
fi
```

This approach gives a clear error message and a deliberate exit code. Using different exit codes for different failure cases makes the script easier to troubleshoot.
### Check whether the account already exists
The next step is to prevent duplicate account creation. `getent passwd` is a reliable way to test for an existing local or directory-backed account because it respects the system's configured name service sources.

```bash
if getent passwd "$1" >/dev/null
then
  echo "User $1 already exists"
  exit 2
fi
```

When an `if` statement tests a command, Bash uses the command's exit status automatically. There is no need to wrap the command in square brackets.
### Read a password securely during execution
Passwords should not appear on the command line because shell history may record them. `read` collects the password during execution instead.

```bash
read -rs -p "Enter a password for $1: " USER_PASSWORD
echo
```

`-r` preserves backslashes, `-s` suppresses screen echo and `-p` prints a prompt. If no variable name is supplied, Bash stores the input in `REPLY`, but an explicit variable name is clearer.

On RHEL, `passwd --stdin` reads a password from standard input. That option is convenient but platform-specific. For broader portability, `chpasswd` is usually safer across distributions.

```bash
echo "$USER_PASSWORD" | passwd --stdin "$1"
```

Reading the password during execution keeps it out of command history and off the process list. That is a direct improvement over supplying it as a second positional parameter.
### Create the account and confirm the result
With input validated and the password collected, the script can create the user and display the resulting entry.

```bash
useradd -m "$1"
echo "$USER_PASSWORD" | passwd --stdin "$1"
getent passwd "$1"
```

`-m` creates the home directory. `getent passwd "$1"` confirms the account details after creation.

The example assumes the script runs with suitable privileges, either through `sudo` inside the command sequence or by running the whole script with administrative rights. User management commands will fail without that access.
## Improving the script with loops and functions
A single-user tool works, but a practical administration script often needs to handle many accounts in one run. Bash loops and functions provide the structure to do that cleanly.
### Process all user names
A `for` loop can iterate over every supplied user name. The choice between `"$*"` and `"$@"` is critical.

When quoted, `"$*"` turns all positional parameters into one string. If the input is `u1 u2 u3`, Bash may treat it as a single combined value. `"$@"`, by contrast, preserves each positional parameter as a separate item. For looping over user names, `"$@"` is the correct choice.

```bash
for u in "$@"
do
  echo "User $u"
done
```

This distinction is small but important. It determines whether the script creates three accounts or tries to process one malformed name. It also preserves quoted multi-word values correctly when the input deliberately contains spaces.
### Require a non-empty password
A password prompt should not accept an empty value. A `while` loop can keep asking until the variable contains text.

```bash
USER_PASSWORD=""
while [ -z "$USER_PASSWORD" ]
do
  read -rs -p "Enter a password for $u: " USER_PASSWORD
  echo
done
```

`-z` is the direct test for an empty string. The loop above is clearer and more accurate than trying to infer emptiness from a negated non-empty test. It ensures the script only continues once the password variable holds real input.

This pattern also shows why validation belongs close to the input step. Bash does not need to set a password, call `passwd` and then discover that the value was empty. It can reject the empty input immediately and ask again.
### Use functions to organise the script
Functions separate tasks into readable blocks. The script becomes easier to maintain because account creation, password collection and reporting each sit in one place.

```bash
create_user() {
  useradd -m "$1"
  getent passwd "$1"
}

set_password() {
  USER_PASSWORD=""
  while [ -z "$USER_PASSWORD" ]
  do
    read -rs -p "Enter a password for $1: " USER_PASSWORD
    echo
  done
  echo "$USER_PASSWORD" | passwd --stdin "$1"
}
```

The main body of the script then becomes much shorter.

```bash
for u in "$@"
do
  create_user "$u"
  set_password "$u"
done
```

This structure improves readability because the loop describes the workflow in plain terms. It also improves debugging because each function has one clear responsibility.

A further refinement allows one password prompt to populate a variable once, then reuse that value for every user created in the same run. That saves time when several accounts need the same initial password. It also demonstrates Bash variable scope in a practical way. A variable set in the script remains available to later function calls unless a separate scope is introduced.
### Confirm the script works
Testing should continue after the script prints success messages. A sensible check is to switch to one of the newly created accounts with `su - username` and verify that the password works. That confirms both account creation and password assignment rather than trusting the display output alone.
## Practical scripting habits
Several habits make Bash scripts more reliable and easier to support.
### Build in stages
The user creation example improves step by step: first argument validation, then existence checks, then password input, then account creation, then loops and functions. This staged approach isolates faults quickly. It also makes it easier to test each feature before adding the next.
### Prefer readable code over clever code
Indentation is not syntactically required in shell scripts, but it makes loops, conditionals and functions much easier to follow. Clear names such as `USER_PASSWORD` or `create_user` communicate intent better than cryptic abbreviations. Small functions make large scripts feel manageable.
### Test commands interactively first
Because scripts mirror the shell, the command line is the best place to verify syntax, exit status and command behaviour. Once the command works interactively, it can move into a script with less risk. The shell's history and shortcuts can accelerate this cycle, but the important habit is disciplined testing.
### Handle portability deliberately
Some commands and options behave differently across distributions. `passwd --stdin` works on Red Hat systems but not everywhere. When portability matters, the script should prefer cross-platform tools or document the platform-specific dependency.
### Separate validation from action
Checks such as argument counting and account existence should happen before `useradd` or `passwd` runs. This keeps failure cases predictable and avoids partial changes.
### Treat scripts as maintainable tools
Even a short script benefits from comments, a consistent style and a clear purpose. The most useful scripts are not merely functional. They are also easy to read a week later, easy to debug under pressure and easy to extend when requirements change.
## Key takeaways
Bash scripting on RHEL is an exercise in disciplined command-line automation. The essential ideas are straightforward:
- shell scripts use the same syntax as interactive shell commands
- exit status drives conditional behaviour
- the shebang selects the correct interpreter
- execute permission and `PATH` determine how a script runs
- special variables make scripts adaptable
- `read` gathers sensitive input more safely than command-line arguments
- `"$@"` is the correct way to loop over positional parameters
- `while` loops handle validation and retry logic
- functions keep repetitive tasks concise and readable