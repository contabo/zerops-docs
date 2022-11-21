# Installation

Zerops **zCLI** is an open-sourced command-line client created using Go language and distributed through NPM or YARN, hosted on GitHub ([github.com/zeropsio/zcli](https://github.com/zeropsio/zcli)), built and published to package managers through a [GitHub Action](https://github.com/zeropsio/zcli/actions?query=workflow%3A%22Upload+build+asset%22). Currently, the **zCLI** is supported on **Linux** (x86 & x64 architecture), **macOS** (x64 & M1 architecture), and **Windows** (x64 architecture only).

<!-- markdownlint-disable DOCSMD004 -->
::: tip Built-in VPN functionality is available on all platforms
 The implementation of a WireGuard-based [VPN](/documentation/cli/vpn.html) built-in protocol is available to use on all platforms for which zCLI exists.
:::
<!-- markdownlint-enable DOCSMD004 -->

## How to install

### Using npm

The Zerops **zCLI** command-line client can be installed using **npm**

```bash
npm i -g @zerops/zcli
```

and it will be available under the **`zcli`** command.

### Download the zCLI static binary

Alternatively, only the **zCLI** binary can be downloaded from [GitHub releases](https://github.com/zeropsio/zcli/releases). You have to select the appropriate version corresponding to the system you want to run it on.

#### macOS x64

```bash
$ wget https://github.com/zeropsio/zcli/releases/latest/download/zcli-darwin-amd64 -O zcli
$ chmod +x zcli

# Modify the system PATH appropriately or use a symlink to run it globally.
# If you want to use it globally, running it just entering the `zcli` command,
# we recommend creating a symbolic link using the following command.

# The command supposes that the current directory is the same directory you downloaded
# the zCLI binary file, and the directory where the symlink will be created
# is a directory already on the system PATH.

$ sudo ln -s $(pwd)/zcli /usr/local/bin/zcli
```

#### macOS M1

```bash
$ wget https://github.com/zeropsio/zcli/releases/latest/download/zcli-darwin-arm64 -O zcli
$ chmod +x zcli

# Modify the system PATH appropriately or use a symlink to run it globally.
# If you want to use it globally, running it just entering the `zcli` command,
# we recommend creating a symbolic link using the following command.

# The command supposes that the current directory is the same directory you downloaded
# the zCLI binary file, and the directory where the symlink will be created
# is a directory already on the system PATH.

$ sudo ln -s $(pwd)/zcli /usr/local/bin/zcli
```

#### Linux x64

```bash
$ wget https://github.com/zeropsio/zcli/releases/latest/download/zcli-linux-amd64 -O zcli
$ chmod +x zcli

# Modify the system PATH appropriately or use a symlink to run it globally.
# If you want to use it globally, running it just entering the `zcli` command,
# we recommend creating a symbolic link using the following command.

# The command supposes that the current directory is the same directory you downloaded
# the zCLI binary file, and the directory where the symlink will be created
# is a directory already on the system PATH.

$ sudo ln -s $(pwd)/zcli /usr/local/bin/zcli
```

#### Linux i386

```bash
$ wget https://github.com/zeropsio/zcli/releases/latest/download/zcli-linux-i386 -O zcli
$ chmod +x zcli

# Modify the system PATH appropriately or use a symlink to run it globally.
# If you want to use it globally, running it just entering the `zcli` command,
# we recommend creating a symbolic link using the following command.

# The command supposes that the current directory is the same directory you downloaded
# the zCLI binary file, and the directory where the symlink will be created
# is a directory already on the system PATH.

$ sudo ln -s $(pwd)/zcli /usr/local/bin/zcli
```

#### Windows x64

```bash
curl https://github.com/zeropsio/zcli/releases/latest/download/zcli-win-x64.exe -L -o zcli.exe

# Modify the system PATH appropriately or use a symlink to run it globally.
# If you want to use it globally, running it just entering the `zcli` command,
# we recommend creating a symbolic link using the following command.

# The command supposes that the current directory is the same directory you downloaded
# the zCLI binary file, and the directory where the symlink will be created
# is a directory already on the system PATH.

mklink %LOCALAPPDATA%\Microsoft\WindowsApps\zcli.exe %CD%\zcli.exe
```
