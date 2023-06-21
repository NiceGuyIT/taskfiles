# Taskfiles

A collection of task files for use with [task][]. There's a repo that has [advanced examples][].

[task]: https://github.com/go-task/task

[advanced examples]: https://gitlab.com/megabyte-labs/common/shared/-/tree/master/.config/taskfiles

## Goals

This project strives to make it easy to run tasks while maintaining a minimal impact to the system. These guidelines are
used to achieve this goal.

1. Minimal requirements to bootstrap the system.
2. Only single binary tools are used. This usually means tools that are written in Rust or Go.
3. Cross-platform support.
4. If possible, configuration values are passed via environmental variables.
5. Streaming is preferred over temporary files.

## Requirements

The following programs are needed to download and uncompress files.

| OS          | Program    | Purpose                                                   |
|-------------|------------|-----------------------------------------------------------|
| Linux/macOS | curl       | Download files from the web                               |
| Linux/macOS | tar        | Extract '.tar.gz' files                                   |
| Linux/macOS | gunzip     | gunzip '.tar.gz' files                                    |
| Linux/macOS | unzip      | unzip '.zip' files                                        |
| Windows     | PowerShell | 'Invoke-WebRequest' cmdlet to download files from the web |
| Windows     | PowerShell | 'Expand-Archive' cmdlet to unzip files                    |
| All*        | git        | Required to `git clone` this repo.                        |

> **Note**
>
> In the future it may be possible to download and uncompress a release, alleviating the need for `git`.

> **Note 2**
>
> Note: [rc-zip][] may be added in the future to allow unzipping from a stream.

[rc-zip]: https://github.com/fasterthanlime/rc-zip

## Install

Follow these steps to bootstrap the system. Actual commands are given below.

1. Clone this repo.
2. Download [task][] into the current directory.
3. Run `./task init:all` to create the `bin` directory and download the bare minimum.
4. Add the `bin` directory to your path.
    - If `task` is run with elevated permissions, the `bin` directory will be a system-side bin directory.
    - If `task` is run by a regular user, the `bin` directory will be in their home directory.
    - `export BIN_DIR=.` to use the current directory. `BIN_DIR` can be set to any directory.

| OS      | User                 | `bin` directory              |
|---------|----------------------|------------------------------|
| Linux   | root                 | `/usr/local/bin`             |
| Linux   | user                 | `$HOME/bin`                  |
| macOS   | root                 | `/usr/local/bin`             |
| macOS   | user                 | `$HOME/bin`                  |
| Windows | Elevated Permissions | `%ProgramData\taskfiles\bin` |
| Windows | user                 | `%USERPROFILE%\bin`          |

### *nix (Linux/macOS)

<details>
  <summary>Install taskfiles on Linux/macOS</summary>

```bash
git clone https://github.com/NiceGuyIT/taskfiles
cd taskfiles

# Download task
os=$(uname -s | tr '[:upper:]' '[:lower:]')
arch=$(uname -m | sed 's/x86_/amd/')
repo="https://github.com/go-task/task/releases/latest/download/task_${os}_${arch}.tar.gz"
curl --location --output - "$repo" | tar -zxf - task
chmod a+x ./task

# Initialize the tasks. This downloads 'task' to the bin directory.
./task init:all

# Cleanup
rm ./task
```

</details>

### Windows

Note: Installing git is outside the scope.

<details>
  <summary>Install taskfiles on Windows</summary>

```PowerShell
git clone https://github.com/NiceGuyIT/taskfiles
cd taskfiles

# Download task
$os = "windows"
$arch = "amd64" # 32-bit not supported
$repo_url = "https://github.com/go-task/task/releases/latest/download/task_${os}_${arch}.zip"
$tmp_file = New-TemporaryFile
Remove-Item -Path $tmp_file
$tmp_dir = New-Item -ItemType Directory -Path $( Join-Path -Path $ENV:Temp -ChildPath $tmp_file.Name )
$zip_file = Join-Path -Path $tmp_dir -ChildPath "task.zip"
$ProgressPreference = "SilentlyContinue"
Invoke-WebRequest -URI $repo_url -OutFile $zip_file
Expand-Archive -Path $zip_file -DestinationPath $tmp_dir
$task = Join-Path -Path $tmp_dir -ChildPath "task.exe"

# Initialize the tasks. This downloads 'task' to the bin directory.
& $task init:all

# Cleanup
Remove-Item -Path $tmp_dir -Recurse
```

</details>

## System specific tasks

- [Tactical RMM][]

[Tactical RMM]: trmm/README.md

## Tasks

`task` uses Go templates to interpret variable names, including environmental variables. The environmental
variable `NAME` is written as `{{.NAME}}` in the template. The `{{` and `}}` denote the start and end of the template.
The period refers to the variable name; without the period it refers to a function name. For example, `{{.ARCH}}` is
the variable that holds the architecture, while `{{ARCH}}` is a function provided by `task` and returns the
architecture. Since curly braces are part of the YAML syntax, they need to be enclosed in quotes or used in a text
block.

In the documentation, names refer to variable names, `NAME`, not the Go template name `{{.NAME}}`.

### Including tasks, subdirectories and deps, preconditions and cmds

Note: This is my understanding of how `task` works. I could be wrong and will update this as I learn more.

To understand how to call or depend on a task in another file, you need to first understand the limitations or drawbacks
of task.

- `task` does not output if a `task`, `deps`, or `precondition` is not found or invalid. This makes troubleshooting
  difficult.
- In order to call tasks in other files, call from the main namespace `:parent:task`.
- To get the output of a task into a variable, use the dynamic version of variable assignment by using `sh` and
  calling the task executable. For example, `task namespace:task-name VALUE="some value"`. See [issue #178][].
    - This format does not work for `deps` since it expects a "task", not "shell output".
    - This format does not work for `preconditions` when the goal is to run the task if it needs to. For example, if the
      task is to download a utility, checking for the existence of the binary will cause the task to fail. The
      workaround is to add the task to the `cmds` list.
- Adding the task as another `cmd` in the `cmds` list works even though it goes against the design concepts of task.
    - Adding the task can be done with `task: :namespace:task-name`.
- Go templates' `{{` and `}}` have to be quoted in yaml, forcing the value to be interpreted as a string.
- Use `base64` to pass JSON between tasks, so you don't have to worry about quoting.
- Variables are interpolated _before_ includes are read. See [issue #454][].
- Vars declared in the [included Taskfile][] have preference over the variables in the including Taskfile!
  If you want a variable in an included Taskfile to be overridable, use the default function:
  `MY_VAR: '{{.MY_VAR | default "my-default-value"}}'`
- Since vars are expanded and env is not, and considering included vars take precedent over including vars, it's
  advisable to set global environmental variables as global variables in the main Taskfile, and then assign the
  environmental variable in the task using the global variable. This provides the ability to change the variable in the
  included (child) Taskfile.

Vars are expanded for Env but Env is not expanded for Vars.
This will work:

```yaml
tasks:
  task-name:
    env:
      ENV1: '{{.VAR1}}'
    vars:
      VAR1: 'Hello'
```

This will not work:

```yaml
tasks:
  task-name:
    env:
      ENV1: 'Hello'
    vars:
      VAR1:
        sh: |
          echo $ENV1
```

[issue #178]: https://github.com/go-task/task/issues/178

[issue #454]: https://github.com/go-task/task/issues/454

[included Taskfile]: https://taskfile.dev/usage/#namespace-aliases

## Coding guidelines (best practices?)

- For double quotes, prefer `{{ quote .MY_VARIABLE }}` over `"{{.MY_VARIABLE}}"` to escape double quotes.
- Likewise with single quotes, prefer `{{ squote .MY_VARIABLE }}` over `'{{.MY_VARIABLE}}'` to escape single quotes.
- Enclose paths in double quotes `"{{ osClean "C:\Program Files\Mozilla Firefox\firefox.exe" }}"` to account for spaces and backslashes.
- See this [SO answer][] for nesting conditionals in Go templates.
- vars are not passed to subtasks automatically; they need to be [passed explicitly][].
- [Boolean][sprig Default Functions]: Use empty string "" for false, non-empty string for true.
- Environmental variables are case-insensitive on Windows and case-sensitive on \*nix. The initial case is preserved on the first call but all uppercase on subsequent calls. See [issue 1229][].

### Multi-line PowerShell commands

In order to use mutliline PowerShell commands from the command line, add an extra return after the code block.
See [Powershell fails to run multi-line commands from stdin][].

```text
          powershell -NonInteractive -NoProfile -NoLogo -InputFormat text -OutputFormat text -Command - << 'EOT'
            if (-not (Test-Path -Path "{{.BIN_DIR}}")) {
              New-Item -Path "{{.BIN_DIR}}" -ItemType Directory | Out-Null
            }

          EOT
```

[SO answer]: https://stackoverflow.com/a/68361609

[passed explicitly]: https://github.com/go-task/task/issues/888#issuecomment-1273264393

[sprig Default Functions]: https://go-task.github.io/slim-sprig/defaults.html

[issue 1229]: https://github.com/go-task/task/issues/1229

[Powershell fails to run multi-line commands from stdin]: https://stackoverflow.com/questions/37417613/powershell-fails-to-run-multi-line-commands-from-stdin

### Modifying existing env vars

Modifying _existing_ environmental variables for tasks is impossible because
the [system env have preference over Taskfile envs][]. The environmental variable will need to be set at the beginning
of each `cmd` that needs the modified env var.

[system env have preference over Taskfile envs]: https://github.com/go-task/task/issues/436#issuecomment-768582760

### Path separators

[sprig path][Sprig's path] documentation mentions that paths are separated by the `os.PathSeparator` variable and
processed by the `path/filesystem` package. Unfortunately, `os.PathSeparator` is not available as a path separator in
Task. Using a forward slash `/` on Windows and passing it through `osClean` works. If a path separator is needed,
`FS_PATH_SEP` can be defined as a variable.

```yaml
    vars:
      FS_PATH_SEP: '{{ if eq OS "windows" }}\{{ else }}/{{ end }}'
```

[sprig path]: https://go-task.github.io/slim-sprig/paths.html

### Joining paths on Windows

To join paths on Windows, joint the paths using `print` and pipe it into `osClean` and add double quotes around teh
template expression. Remember, the underlying shell is bash calling PowerShell. Piping `print` or `osClean` to `quote`
will double the backslashes `\`. Run the `testing:join-path` task to see this in action.

```yaml
      - cmd: |
          echo PowerShell osClean test: "{{ print .BIN_PATH .FS_PATH_SEP .PATH2 .FS_PATH_SEP "maintenanceservice.log" | osClean }}"
          powershell -NonInteractive -NoProfile -NoLogo -InputFormat text -OutputFormat text -Command '
            $file="{{ print .BIN_PATH .FS_PATH_SEP .PATH2 .FS_PATH_SEP "maintenanceservice.log" | osClean }}"
            Write-Output "File: $file"
          '
```

## Enhancements

- The [gh cli][] requires authentication for all actions. It's highly unlikely to be supported.
- [ghrel][] is an alternative to download files from GitHub. If it didn't download to the current directory, this may be a good alternative.
- [just][] is a command runner. Don't know if it will be useful.

[gh cli]: https://github.com/cli/cli

[ghrel]: https://github.com/jreisinger/ghrel

[just]: https://github.com/casey/just
