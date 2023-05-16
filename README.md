ask files for use with [task][]. There's a repo that has [advanced examples][].

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

- [GitHub][]
- [Tactical RMM][]

[GitHub]: github/README.md
[Tactical RMM]: trmm/README.md

## Tasks

`task` uses Go templates to interpret variable names, including environmental variables. The environmental
variable `NAME` is written as `{{.NAME}}` in the template. The `{{` and `}}` denote the start and end of the template.
The period refers to the variable name; without the period it refers to a function name. For example, `{{.ARCH}}` is
the variable that holds the architecture, while `{{ARCH}}` is a function provided by `task` and returns the
architecture. Since curly braces are part of the YAML syntax, they need to be enclosed in quotes or used in a text
block.

In the documentation, names refer to variable names, `NAME`, not the Go template name `{{.NAME}}`.

## Including tasks, subdirectories and deps, preconditions and cmds

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

[issue #178]: https://github.com/go-task/task/issues/178

## Coding guidelines (best practices?)

- For double quotes, prefer `{{ quote .MY_VARIABLE }}` over `"{{.MY_VARIABLE}}"` to escape double quotes.
- Likewise with single quotes, prefer `{{ squote .MY_VARIABLE }}` over `'{{.MY_VARIABLE}}'` to escape single quotes.
- See this [SO answer][] for nesting conditionals in Go templates.
- vars are not passed to subtasks automatically; they need to be passed explicitly.


[SO answer]: https://stackoverflow.com/a/68361609
[passed explicitly]: https://github.com/go-task/task/issues/888#issuecomment-1273264393
