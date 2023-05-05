# Taskfiles

A collection of task files for use with [task][]. There's a repo that has [advanced examples][].

[task]: https://github.com/go-task/task

[advanced examples]: https://gitlab.com/megabyte-labs/common/shared/-/tree/master/.config/taskfiles

## Goals

This project strives to make it easy to run tasks while maintaining a minimal impact to the system. These guidelines are
used to achieve this goal.

1. Minimal requirements to bootstrap the system.
2. Only single binary tools are used. This usually means tools that are written in Rust or Go.
3. All configuration is passed via environmental variables.
4. Streaming is preferred over temporary files.

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

\*In the future it may be possible to download and uncompress a release, alleviating the need for `git`.

Note: [rc-zip][] may be added in the future to allow unzipping from a stream.

[rc-zip]: https://github.com/fasterthanlime/rc-zip

## Install

Follow these steps to bootstrap the system. Actual commands are given below.

1. Clone this repo.
2. Download [task][] into the current directory.
3. Run `./task init-all` to create the `bin` directory and download the bare minimum.
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
task init --status

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
& $task init --status

# Cleanup
Remove-Item -Path $tmp_dir -Recurse
```

</details>

## Tasks

`task` uses Go templates to interpret variable names, including environmental variables. The environmental
variable `NAME` is written as `{{.NAME}}` in the template. The `{{` and `}}` denote the start and end of the template.
The period refers to the variable name; without the period it refers to a function name. For example, `{{.ARCH}}` is
the variable that holds the architecture, while `{{ARCH}}` is a function provided by `task` and returns the
architecture. Since curly braces are part of the YAML syntax, they need to be enclosed in quotes or used in a text
block.

The names refer to variable names, `NAME`, not the Go template name `{{.NAME}}`.

