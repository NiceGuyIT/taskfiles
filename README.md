# taskfiles

A collection of task files for use with [task][].

[task]: https://github.com/go-task/task

## Install

1. Clone this repo.
2. Download [task][] into the current directory.
3. Run `task init` to create the `bin` directory and download the bare minimum.
4. Add the `bin` directory to your path. If `task` is run with elevated permissions, the `bin` directory will be a
   system-side bin directory. If `task` is run by a regular user, the `bin` directory will be in their home directory.

| OS      | User                 | `bin` directory              |
|---------|----------------------|------------------------------|
| Linux   | root                 | `/usr/local/bin`             |
| Linux   | user                 | `$HOME/bin`                  |
| macOS   | root                 | `/usr/local/bin`             |
| macOS   | user                 | `$HOME/bin`                  |
| Windows | Elevated Permissions | `%ProgramData\taskfiles\bin` |
| Windows | user                 | `%USERPROFILE%\bin`          |

### *nix (Linux/macOS)

```bash
git clone https://github.com/NiceGuyIT/taskfiles
cd taskfiles

# Download task
os=$(uname -s | tr '[:upper:]' '[:lower:]')
arch=$(uname -m | sed 's/x86_/amd/')
repo="https://github.com/go-task/task/releases/latest/download/task_${os}_${arch}.tar.gz"
curl --location --output - "$repo" | tar -zxf - task
chmod a+x ./task

# Initialize the tasks
task init

# Cleanup
rm ./task
```

### Windows

Note: Installing git is outside the scope.

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

# Initialize the tasks
& $task init --status

# Cleanup
Remove-Item -Path $tmp_dir -Recurse
```
