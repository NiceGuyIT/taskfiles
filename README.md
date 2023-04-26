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

# Initialize the tasks
task init

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

# Initialize the tasks
& $task init --status

# Cleanup
Remove-Item -Path $tmp_dir -Recurse
```

</details>

## Tasks

### GitHub

#### Download

The `github:download` task will download executables from GitHub repos.

- ENV vars: _Source_; Meaning or explanation.
- `.NAME`: _Parameter_; Name of the binary to download. Normally this is the repo name.
- `.OWNER`: _Parameter_; Owner of the repo.
- `.ASSET_PATTERN`: _Parameter_; Pattern of the filename listed in the release assets.
- `.COMPRESS_EXT`: _Parameter_, _Generated_; Extension for the compression. Defaults to "tar.gz" on \*nix and "zip" on
  Windows. Use "None" to indicate there is no compression.
- `.ARCH`: _Parameter_; Architecture to download. Some releases use "x86_64" instead of "amd64".
- `.REPO`: _Parameter_, _Generated_; Default to `.NAME/.OWNER` if not provided.
- `.FILE_NAME`: _Generated_; `.NAME.exeExt`
- `.BIN_NAME`: _Generated_; `.BIN_DIR/.FILE_NAME`
- `.REPO_URL`: _Generated_; `https://api.github.com/repos/{{.REPO}}/releases/latest`

## Get Attribute

The `github:get-attribute` task is an internal task that gets attributes from GitHub's API that describe the asset.
This is used to get the filename and download URL.

- ENV vars: _Source_; Meaning or explanation.
- `.REPO_URL`: _Parameter_; GitHub API repository URL.
- `.ASSET_PATTERN`: _Parameter_; Pattern to search for to match an asset.
- `.ATTRIBUTE`: _Parameter_; Attribute to get.
