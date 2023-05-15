# Tactical RMM Tasks

## Variable names

- Global variables are used as default values and prefixed with `TRMM_DEFAULT_`.
- The community scripts repo is referred to as 'COMMUNITY_*' to distinguish from "my scripts" variables.
  - `TRMM_DEFAULT_COMMUNITY_PATH` is the full path to the community scripts repo.
  - `TRMM_DEFAULT_COMMUNITY_JSON` is the name of the community scripts JSON file.
- The custom scripts repo is referred to as 'MY_SCRIPTS' to distinguish from the community scripts variables.
  - `MY_SCRIPTS_PATH` is the full path to the MY_SCRIPTS repo.
  - `MY_SCRIPTS_NAME_PREFIX` is prepended to the script's name. All MY_SCRIPTS will have this prefix.
  - `MY_SCRIPTS_DIR_PREFIX` is the directory inside the repo to export MY_SCRIPTS.
  - `MY_SCRIPTS_DIR` is the full path to the 'MY_SCRIPTS' directory inside the repo.
  - `MY_SCRIPTS_JSON` is the full path to the script definition file for MY_SCRIPTS.

## Custom Scripts Import

The `trmm:scripts-repo-custom-scripts-import` task merge custom scripts with the community scripts and then import them
into TRMM. Use this to update your scripts in TRMM.

## Custom Scripts Export and Import

The `trmm:scripts-repo-custom-scripts-export-import` task will export the custom scripts from the database, save them
in a directory, merge them with the community script and import them back into TRMM. Use this for the initial
export/import process.

## Scripts Repo Reset

The `trmm:scripts-repo-reset` task will reset the community scripts repo. This is mostly an internal task to undo
changes to the `community_scripts.json` that is modified by other tasks, but can also be used to reset any changes.

## Scripts Repo Update

The `trmm:scripts-repo-update` task will update the community scripts from GitHub.

## Database Export All

The `trmm:scripts-database-export-all` task will export all custom scripts from the database into a directory of
scripts and the script definitions to the JSON file.

## Database Export To File

The `trmm:scripts-database-export-to-file` task will save the JSON exported from the database into a file and append
to the JSON definition file. The definition file is in the same format as the community scripts JSON.
=======
    - `TRMM_DEFAULT_COMMUNITY_PATH` is the full path to the community scripts repo.
    - `TRMM_DEFAULT_COMMUNITY_JSON` is the name of the community scripts JSON file.
- The custom scripts repo is referred to as 'MY_SCRIPTS' to distinguish from the "community scripts" variables.
    - `MY_SCRIPTS_PATH` is the full path to the MY_SCRIPTS repo.
    - `MY_SCRIPTS_NAME_PREFIX` is prepended to the script's name. All MY_SCRIPTS will have this prefix.
    - `MY_SCRIPTS_DIR_PREFIX` is the directory inside the repo to export MY_SCRIPTS.
    - `MY_SCRIPTS_DIR` is the full path to the 'MY_SCRIPTS' directory inside the repo.
    - `MY_SCRIPTS_JSON` is the full path to the script definition file for MY_SCRIPTS.

## My Scripts Export and Import

The `trmm:my-scripts-export-import` task will export my scripts from the database, save them in a directory, merge them
with the community repo and import them back into TRMM. Use this for the initial export/import process.

## My Scripts Import

The `trmm:my-scripts-import` task merges my scripts with the community repo and imports them into TRMM. Use this to
update your scripts in TRMM.

## My Scripts Export

The `trmm:my-scripts-export` task will export the custom scripts from the database into a directory and add the
script definition to the JSON file. Use NAME_PATTERN to limit the export to the matching pattern.

> Note: This task does not check for duplicates in the JSON file. Use `git` to merge the "new" changes to your repo.

## My Scripts Database Export To File

The `trmm:my-scripts-database-export-to-file` task will save the JSON exported from the database into a file and append
to the JSON definition file. The definition file is in the same format as the community scripts JSON.

> Note: This task is used by the other tasks and not meant to be called directly.

## Community Repo Reset

The `trmm:community-repo-reset` task will reset the community scripts repo. This is mostly an internal task to undo
changes to the `community_scripts.json` that is modified by other tasks, but can also be used to reset any changes.

## Community Repo Update

The `trmm:scripts-repo-update` task will update the community scripts repo from GitHub. A reset is performed first to
prevent merge conflicts.

## Config Get Value

The `trmm:config-get-value` task is a helper task to get the configuration value from the TRMM `local_settings.py` file.
The CONFIG_KEY is the Python variable you want to get, with an optional JSON_EXP that is applied to extract values
within the CONFIG_KEY.

## config Get Tactical User

The `trmm:config-get-tactical-user` will get the OS running TRMM as determined by the ownership of `local_settings.py`.

## Management Command

The `trmm:management-command` task can run TRMM management commands from root or the `tactical` user. If run from root,
the commands are wrapped around `su`.

## Management Command: Load Community Scripts

The `trmm:management-command-load-community-scripts` task will load the community scripts into TRMM.
