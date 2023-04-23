# taskfiles
A collection of task files for use with [task][].

[task]: https://github.com/go-task/task

## Install

1. Clone this repo.
2. Download [task][] into the current directory.
3. Run `task init` to create the `bin` directory and download the bare minimum.

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
$tmp_dir = New-Item -ItemType Directory -Path $(Join-Path -Path $ENV:Temp -ChildPath $tmp_file.Name)
$zip_file = Join-Path -Path $tmp_dir -ChildPath "task.zip"
$ProgressPreference = "SilentlyContinue"
Invoke-WebRequest -URI $repo_url -OutFile $zip_file
Expand-Archive -Path $zip_file -DestinationPath $tmp_dir
$task = Join-Path -Path $tmp_dir -ChildPath "task.exe"

# Initialize the tasks
& $task init --status

Remove-Item -Path $tmp_dir -Recurse
```
