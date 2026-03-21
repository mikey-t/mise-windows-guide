# Mise Windows Setup Supplemental Info

This document is for supplemental info about installing mise on Windows and WSL (Ubuntu). For the main guide, go back to the [./README.md](./README.md).

- [Mise Windows Setup Supplemental Info](#mise-windows-setup-supplemental-info)
- [Configuration Setting Explanations](#configuration-setting-explanations)
  - [Idiomatic Version File Settings](#idiomatic-version-file-settings)
- [Shims](#shims)
  - [Windows Shim Mode](#windows-shim-mode)
  - [Shims vs PATH](#shims-vs-path)
- [Installing Global npm Packages](#installing-global-npm-packages)
- [Misc Mise Commands](#misc-mise-commands)


# Configuration Setting Explanations

For more explanation of the "idiomatic version file" setting see: https://mise.jdx.dev/configuration.html#idiomatic-version-files.

## Idiomatic Version File Settings

Allow mise to look in standard file locations for tool versions instead of forcing the need to rely on a dedicated mise config file (i.e. `mise.toml`).

The commands to enable this for Node.js, npm and pnpm:

```bash
mise settings add idiomatic_version_file_enable_tools node
mise settings add idiomatic_version_file_enable_tools npm
mise settings add idiomatic_version_file_enable_tools pnpm
```

For Node.js, this allows mise to look in the following locations:

- `package.json` - `devEngines.runtime`
- `.node-version`
- `.nvmrc`

And for npm and pnpm, it allows mise to look in `package.json` field `devEngines.packageManager` for the package manager and version.

If you're using these settings, you don't need to have a `mise.toml` file in your projects.

For an example package.json snippet, see section in main readme: [Package.json Snippet](./README.md#packagejson-snippet).

# Shims

## Windows Shim Mode

This setting configures mise to use "exe" shims, which means an exe will be generated when adding or updating a tool through mise:
```bash
mise settings set windows_shim_mode exe
```

You also need to ensure that the mise shims directory is in the system PATH:

```
%LOCALAPPDATA%\mise\shims
```

Without this setting, attempting to spawn a process to execute a tool within Node.js would result in a "command not found" error. You would be forced you to use the `shell` option which utilizes `.cmd`, which would in turn cause a deprecation warning in newer versions of Node.js, or you would have to use some other workaround. With this setting in place, you can simply spawn the command. For example: `spawn('npm', ['-v'])` will find the command with no workarounds in both Windows and WSL.

Mise should automatically handle this, but you can tell it to regenerate shims manually by running `mise reshim` (or `mise reshim --force` to delete all shims before regenerating). See https://mise.jdx.dev/cli/reshim.html.

## Shims vs PATH

See official docs on Shims vs PATH: https://mise.jdx.dev/dev-tools/shims.html#shims-vs-path.

It's noteworthy that on Windows if you only setup mise context activation with "mise activate" in powershell in your profile (the "PATH" option), your interactive shell session will work fine, but spawning mise managed tools from Node.js will result in a "command not found" error.

However, you don't want to change to using "mise activate --shims" in your powershell profile because then you miss out on the advantages of the PATH approach.

Instead of choosing one or the other, I'm using the PATH approach in my powershell profile, then setting shim mode to "exe" (instead of default "file" mode), and then manually adding the mise shims directory to the system PATH so that tool exe shims are found when Node.js tries to spawn them with something like `spawn('npm', ['-v'])`.

Some interesting links to related github discussions, the PR where exe shims functionality was introduced and the release notes for when that PR was merged:

- "The node shim created by mise is prone to errors when spawning files on Windows. #4773": https://github.com/jdx/mise/discussions/4773
- "Use executable Windows shims for mise dev tools - Proof-of-concept, ready for use in local dev env #7998": https://github.com/jdx/mise/discussions/7998
- PR "feat(shim): add native .exe shim mode for Windows #8045": https://github.com/jdx/mise/pull/8045
- Release "v2026.2.7: Windows Gets Real" (2026-02-08): https://github.com/jdx/mise/releases/tag/v2026.2.7

# Installing Global npm Packages

Installing global npm packages with mise instead of pnpm is great because mise can
generate exe shims while pnpm cannot (assuming you followed this guide for setup - see [Shims](#shims) section for more info). With mise, spawning tool commands will work in Node.js
instead of failing with a "command not found" error on Windows.

This functionality utilizes the mise "npm backend". To utilize pnpm instead of npm for the "npm backend" package manager, first configure your mise installation with command: `mise settings set npm.package_manager pnpm`.

Example global install of swig-cli (note that you still use "npm" in the command, even if you've configured the mise "npm backend" to use pnpm):

```bash
mise use -g npm:swig-cli@latest
```

For more info on mise backends, see:
- https://mise.jdx.dev/dev-tools/backends/#backends
- https://mise.jdx.dev/dev-tools/backends/npm.html

# Misc Mise Commands

Below are some misc commands I've found useful.

For all commands, see docs: https://mise.jdx.dev/cli/.

```bash
# Show mise version (note that "-v" doesn't work)
mise version

# Update mise - windows manually installed version
mise self-update

# Update mise - Ubuntu apt installed version
sudo apt update && sudo apt install --only-upgrade mise

# Show versions of tools that mise recognizes from config settings for the current project
mise ls

# Set node version for a project by adding to mise config file.
# Don't use this if utilizing package.json instead of mise.toml.
mise use --pin node@24

# Install a node version globally
mise use --global node@24

# Show available node versions to download.
# Note that this doesn't always seem to be completely up to date - see Node's
# site for up-to-date list: https://nodejs.org/en/about/previous-releases
mise ls-remote node

# Install tools in project based on config settings
mise install

# Runs with tool with version that mise detects should be used for the project.
# Note that the docs have syntax that doesn't work in powershell, so below are two
# separate examples.
# Nix:
mise exec -- node my-script.js
# Windows:
mise exec --command "node my-script.js"

# Install but don't activate:
mise install node@version

# Uninstall - removes tool version from system but doesn't modify config. Note that
# this is different than the "unuse" command.
mise uninstall node@version

# Return path to installed
mise where node@version

# Shows path of config file(s) used and tool versions, for example "%USERPROFILE%\.config\mise\config.toml"
mise config ls

# Remove a global tool version from global config. Note that this is different
# than "uninstall". Omit the "-g" for updating local project config to remove tool.
mise unuse -g node@version
```
