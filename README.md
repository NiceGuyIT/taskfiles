# taskfiles

A collection of task files for use with [task][]. There's a repo that has [advanced examples][].

[task]: https://github.com/go-task/task

[advanced examples]: https://gitlab.com/megabyte-labs/common/shared/-/tree/master/.config/taskfiles

## Goals

This project strives to make it easy to run tasks while maintaining a minimum impact to the system.

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
| All         | git        | Required to `git clone` this repo.                        |

Note: [rc-zip][] may be added in the future to allow unzipping from a stream.

[rc-zip]: https://github.com/fasterthanlime/rc-zip

## Install

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

`task` uses Go templates to interpret variable names, including environmental variables. The environmental
variable `NAME` is written as `{{.NAME}}` in the template. The `{{` and `}}` denote the start and end of the template.
The period refers to the variable name; without the period it refers to a function name. For example, `{{.ARCH}}` is
the variable that holds the architecture, while `{{ARCH}}` is a function provided by `task` and returns the
architecture. Since curly braces are part of the YAML syntax, they need to be enclosed in quotes or used in a text
block.

The names below refer to variable names, `NAME`, not the Go template name `{{.NAME}}`.

### GitHub Download

The `github-download` task will download executables from GitHub repos. Here is the list of supported variables.

> ⚠️ **Note:** The `ASSET_PATTERN` is used to find 1 asset from the GitHub API. `ASSET_PATTERN` is normally
> something like `{{.NAME}}_*_{{OS}}_{{.ARCH}}{{.COMPRESS_EXT}}'` where `COMPRESS_EXT` is `.zip`
> or `.tar.gz`. `COMPRESS_EXT` is mandatory if the asset has an extension. That's because some assets have an extension
> while others don't.
---

| ENV var         | Parameter | Generated | Default                       | Meaning or explanation                                                               |
|-----------------|:---------:|:---------:|-------------------------------|--------------------------------------------------------------------------------------|
| `NAME`          | &#x2714;  |           |                               | Required. Name of the repo. Also the name of the binary to download.                 |
| `OWNER`         | &#x2714;  |           |                               | Required. Owner of the repo.                                                         |
| `COMPRESS_EXT`  | &#x2714;  |           |                               | Optional. Compression extension. Usually `.tar.gz` for \*nix and `.zip` for Windows  |
| `ASSET_PATTERN` | &#x2714;  |           |                               | Required. Pattern of the filename listed in the release assets.                      |
| `ARCH`          | &#x2714;  |           |                               | Architecture to download. Some releases use "x86_64" instead of "amd64".             |
| `REPO`          | &#x2714;  | &#x2714;  | `NAME/OWNER`                  | Repository name.                                                                     |
| `FILE_NAME`     | &#x2714;  | &#x2714;  | `NAME.exeExt`                 | Use this if the binary is not the same as the NAME.                                  |
| `PACKAGE_DIR`   | &#x2714;  | &#x2714;  | `ASSET_NAME` - `COMPRESS_EXT` | Use this if the package directory inside the archive is different than `ASSET_NAME`. |
| `BIN_NAME`      |           | &#x2714;  | `BIN_DIR/FILE_NAME`           |                                                                                      |
| `REPO_URL`      |           | &#x2714;  |                               | `https://api.github.com/repos/{{.REPO}}/releases/latest`                             |

### GitHub Download Asset

The `github-download-asset` task is an internal task that will download a release, uncompress if necessary, and save the
binary in `.BIN_NAME`.

- ENV vars: _Source_; Meaning or explanation.
- `.PACKAGE_NAME`: _Parameter_; Package name as extracted from the API.
- `.COMPRESS_EXT`: _Parameter_; Compression extension as determined from `.PACKAGE_NAME`.
- `.PACKAGE_DIR`: _Parameter_; If compressed, the directory inside the compressed package. This is
  normally `.PACKAGE_NAME` minus `.COMPRESS_EXT`.
- `.ASSET_DIR`: _Parameter_; True if the package is compressed and has a subdirectory. Empty otherwise.
- `.DOWNLOAD_URL`: _Parameter_; Full URL to download the asset.
- `.FILE_NAME`: _Parameter_; If compressed, the filename of the file to extract.
- `.BIN_NAME`: _Parameter_; The full path name to the binary file.
- `.BIN_DIR`: _Parameter_; The directory to save the file to.

### GitHub Get Attribute

The `github-get-attribute` task is an internal task that gets attributes from GitHub's API that describe the asset.
This is used to get the filename and download URL.

- ENV vars: _Source_; Meaning or explanation.
- `.REPO_URL`: _Parameter_; GitHub API repository URL.
- `.ASSET_PATTERN`: _Parameter_; Pattern to search for to match an asset.
- `.ATTRIBUTE`: _Parameter_; Attribute to get.
