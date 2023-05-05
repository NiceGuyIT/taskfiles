# GitHub Tasks

## GitHub Download

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

## GitHub Download Asset

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

## GitHub Get Attribute

The `github-get-attribute` task is an internal task that gets attributes from GitHub's API that describe the asset.
This is used to get the filename and download URL.

- ENV vars: _Source_; Meaning or explanation.
- `.REPO_URL`: _Parameter_; GitHub API repository URL.
- `.ASSET_PATTERN`: _Parameter_; Pattern to search for to match an asset.
- `.ATTRIBUTE`: _Parameter_; Attribute to get.
