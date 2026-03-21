# Mise Windows Guide

This is an opinionated [mise](https://mise.jdx.dev) setup guide focusing on Node.js on Windows and WSL (Ubuntu).

Use two separate mise installations for the Windows host and WSL distro. The WSL instructions in this doc are for Ubuntu 24.

Table of Contents:
- [Mise Windows Guide](#mise-windows-guide)
- [Mise Docs](#mise-docs)
- [Windows Setup](#windows-setup)
  - [Do Not](#do-not)
  - [Windows Install](#windows-install)
  - [Windows Post-Install Configuration](#windows-post-install-configuration)
- [WSL Setup - Ubuntu 24](#wsl-setup---ubuntu-24)
  - [Ubuntu Install](#ubuntu-install)
  - [Ubuntu Post-Install Configuration](#ubuntu-post-install-configuration)
  - [Ubuntu Mise Update](#ubuntu-mise-update)
- [Install Tools](#install-tools)
- [Install Global npm Packages](#install-global-npm-packages)
- [Package.json Snippet](#packagejson-snippet)
- [Misc Mise Notes](#misc-mise-notes)
- [Known Issues and Troubleshooting](#known-issues-and-troubleshooting)
- [Supplemental Info](#supplemental-info)

# Mise Docs

- Docs home: https://mise.jdx.dev
- CLI docs: https://mise.jdx.dev/cli/
- Github: https://github.com/jdx/mise

# Windows Setup

## Do Not

- Do not install with Chocolatey. The admin location breaks some functionality.
- Do not install with Scoop. The Scoop repo popularity and maintenance pattern don't inspire confidence, and there are open github issues in the mise repo describing issues with the Scoop install.
- Do not use "Legacy Powershell" (PowerShell 5.1) - use Powershell Core

## Windows Install

- Download windows release (mise-VERSION-windows-x64.zip): https://github.com/jdx/mise/releases
- Extract to: `%USERPROFILE%\.local\mise`
- Add to system PATH: `%USERPROFILE%\.local\mise`
- Add to powershell core profile: `(&mise activate pwsh) | Out-String | Invoke-Expression`
- Refresh shell: `. $Profile`
- Verify:
  - `mise version`
  - `mise doctor`
- Update: `mise self-update`

## Windows Post-Install Configuration

- Configure mise to look in standard files for tool info:
  - `mise settings add idiomatic_version_file_enable_tools node`
  - `mise settings add idiomatic_version_file_enable_tools npm`
  - `mise settings add idiomatic_version_file_enable_tools pnpm`
- Use native Windows shims: `mise settings set windows_shim_mode exe`
- Add shims to system PATH: `%LOCALAPPDATA%\mise\shims`
- Use pnpm for the "npm backend": `mise settings set npm.package_manager pnpm`
- If you added the mise activation to your legacy powershell profile (or if you're using a shared powershell profile that applies to both legacy powershell and powershell core), you can suppress the legacy powershell mise warning with an environment variable: `[Environment]::SetEnvironmentVariable("MISE_PWSH_CHPWD_WARNING", "0", "User")`
- [Install Tools](#install-tools) (see linked section)
- [Install Global npm Packages](#install-global-npm-packages) (see linked section)

# WSL Setup - Ubuntu 24

## Ubuntu Install

Use "Debian/Ubuntu(apt)" instead of "Linux/macOS" from the official docs:
```bash
sudo apt update -y && sudo apt install -y curl
sudo install -dm 755 /etc/apt/keyrings
curl -fSs https://mise.jdx.dev/gpg-key.pub | sudo tee /etc/apt/keyrings/mise-archive-keyring.asc 1> /dev/null
echo "deb [signed-by=/etc/apt/keyrings/mise-archive-keyring.asc] https://mise.jdx.dev/deb stable main" | sudo tee /etc/apt/sources.list.d/mise.list
sudo apt update -y
sudo apt install -y mise
```

Add to `~/.bashrc` (replace `<your_username>`):

```bash
# Mise activation
eval "$(mise activate bash)"

# Ignore windows config when in windows mount user profile directory tree
export MISE_IGNORED_CONFIG_PATHS=/mnt/c/Users/<your_username>/.config/mise/config.toml
```

Refresh shell: `source ~/.bashrc`

Verify install:
- `mise version`
- `mise doctor`

## Ubuntu Post-Install Configuration

- Configure mise to look in standard files for tool info:
  - `mise settings add idiomatic_version_file_enable_tools node`
  - `mise settings add idiomatic_version_file_enable_tools npm`
  - `mise settings add idiomatic_version_file_enable_tools pnpm`
- Use pnpm for the "npm backend": `mise settings set npm.package_manager pnpm`
- [Install Tools](#install-tools) (see linked section)
- [Install Global npm Packages](#install-global-npm-packages) (see linked section)

## Ubuntu Mise Update

**Important**: `mise self-update` won't work in Ubuntu because `apt` was used for the install, so use `apt` to update mise:

```bash
sudo apt update && sudo apt install --only-upgrade mise
```

# Install Tools

- Setup an initial global node version: `mise use -g node@24`
- Setup pnpm globally:
  - `mise use -g pnpm@10.32.1`
  - `pnpm setup`

# Install Global npm Packages

Install any desired global npm packages using mise and its [npm backend](https://mise.jdx.dev/dev-tools/backends/npm.html) instead of npm or pnpm. To ensure mise uses pnpm under the hood instead of npm, be sure to first run `mise settings set npm.package_manager pnpm` if you didn't already run it during the install steps.

For example, to install [swig-cli](https://github.com/mikey-t/swig), run `mise use -g npm:swig-cli@latest`. Note that you still use "npm" in the command even if you've configured the npm backend to use pnpm. To remove the global npm package, use a command like `mise rm -g npm:swig-cli` (include `@version` on the end where needed).

# Package.json Snippet

Example `package.json` snippet to allow mise to determine what versions of node and pnpm to use in a Node.js project (replace versions):

```json
{
  "devEngines": {
    "packageManager": {
      "name": "pnpm",
      "version": "^10.32.1",
      "onFail": "ignore"
    },
    "runtime": {
      "name": "node",
      "version": "24.14.0"
    }
  }
}
```

# Misc Mise Notes

To suppress non-error output, use the `-q` flag to most commands. See https://mise.jdx.dev/cli/#q-quiet.

# Known Issues and Troubleshooting

On Windows/powershell, `mise exec` (alias `mise x`) commands need to use the `--command` param explicitly instead of double dash "--" format and the command itself should also be in quotes. For example, this works in WSL: `mise x node@24 -- node -v`, but not in powershell, which requires this instead: `mise x node@24 --command 'node -v'`. More info on exec command: https://mise.jdx.dev/cli/exec.html.

# Supplemental Info

See [./SupplementalInfo.md](./SupplementalInfo.md)
