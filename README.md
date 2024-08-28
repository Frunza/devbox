# Devbox

## Motivation

Although it is a good idea to run your stuff inside a docker container, there are times when you want to do some local development, or use modern `CLI` tools on your local machine. As you probably know, it usually is very difficult to use more versions of a tool, or switch between versions.

For example, one application might use an older python version and you local python does not work correctly with the one used in the project. What to do?

It would be cool, if we could somehow get the tools we want from some environment, and have the posibbility to just use them. `devbox` addresses this issue.

## Prerequisites

A Linux or MacOS machine for local development. If you are running Windows, you first need to set up the *Windows Subsystem for Linux (WSL)* environment.

You must have [devbox](https://www.jetify.com/devbox/docs/) installed.

## Playing around

First of all, create a new `devbox` environment:
```sh
devbox init
```
This weill create a `devbox.json` file. There is not much there. If you open it, you will see something like:
```sh
{
    "$schema": "https://raw.githubusercontent.com/jetify-com/devbox/0.12.0/.schema/devbox.schema.json",
    "packages": [],
    "shell": {
      "init_hook": [
        "echo 'Welcome to devbox!' > /dev/null"
      ],
      "scripts": {
        "test": [
          "echo \"Error: no test specified\" && exit 1"
        ]
      }
    }
  }
```
From what we can spot right away, the `packages` parameter looks interesting. This is where the tools we want to use are set up.

Let's say we want a specific python version. Since we do not know the syntax yet, we can use the command below to add the latest python package:
```sh
devbox add python
```
If we investigate the `devbox.json` file now, we will notice that `python` was added to the packages:
```sh
  "packages": ["python@latest"],
```
That's probably not what we want, since it is a good idea to pin everything, but at least the syntax is now clear. We will also notice that the dependencies were resolved in a new file named `devbox.lock`
```sh
{
  "lockfile_version": "1",
  "packages": {
    "python@latest": {
      "last_modified": "2024-08-14T11:41:26Z",
      "plugin_version": "0.0.3",
      "resolved": "github:NixOS/nixpkgs/0cb2fd7c59fed0cd82ef858cbcbdb552b9a33465#python3",
      "source": "devbox-search",
      "version": "3.12.4",
      "systems": {
        "aarch64-darwin": {
          "outputs": [
            {
              "name": "out",
              "path": "/nix/store/xnm9sbp02pp2704dvx4kb8g0yxn263pn-python3-3.12.4",
              "default": true
            }
          ],
          "store_path": "/nix/store/xnm9sbp02pp2704dvx4kb8g0yxn263pn-python3-3.12.4"
        },
        "aarch64-linux": {
          "outputs": [
            {
              "name": "out",
              "path": "/nix/store/wviq85xs5awfp75mj5sjq9mn1nkqhq12-python3-3.12.4",
              "default": true
            },
            {
              "name": "debug",
              "path": "/nix/store/cglyj6nr5rcr0j9c36xqvybk5rj7hi6x-python3-3.12.4-debug"
            }
          ],
          "store_path": "/nix/store/wviq85xs5awfp75mj5sjq9mn1nkqhq12-python3-3.12.4"
        },
        "x86_64-darwin": {
          "outputs": [
            {
              "name": "out",
              "path": "/nix/store/dc7kp6g4s76jlfjhjdpibjfc8pglhlcs-python3-3.12.4",
              "default": true
            }
          ],
          "store_path": "/nix/store/dc7kp6g4s76jlfjhjdpibjfc8pglhlcs-python3-3.12.4"
        },
        "x86_64-linux": {
          "outputs": [
            {
              "name": "out",
              "path": "/nix/store/04gg5w1s662l329a8kh9xcwyp0k64v5a-python3-3.12.4",
              "default": true
            },
            {
              "name": "debug",
              "path": "/nix/store/w2ycb82p1q4xdpm2pzcqk7k58f89wv6l-python3-3.12.4-debug"
            }
          ],
          "store_path": "/nix/store/04gg5w1s662l329a8kh9xcwyp0k64v5a-python3-3.12.4"
        }
      }
    }
  }
}
```
We notice here that `python` version `3.12.4` was installed.
Now we can replace `latest` with `3.12.4` in the packages. Actually, let's change this to a previous version, just to make sure everything works as expected. Let's set `python` to version `3.12.3` in the `devbox.json` file:
```sh
  "packages": ["python@3.12.3"],
```

Time for some testing. Let's run
```sh
python --version
```
Depending on your environment, this will most likely return a different version, or `python` might not even be installed.

To use the tools set up in `devbox`, you must run the following command:
```sh
devbox shell
```
This will start a devbox shell, with the tools and environment variables from your local system, overwritten by the tools set up in `devbox`.
First of all, run
```sh
python --version
```
again. The configured version of `3.12.3` should now show up. If you now take a look at the `devbox.lock` file, you will notice that the `3.12.3` version of `python` is set up. So the `devbox.lock` file is updated every time we run the
```sh
devbox shell
```
command. Therefore it is a good idea to pin all packages, so that we do not end up with unexpected new versions that work differently.

You can use `devbox` with different configuration for different projects, or even try to update your default shell with tools that you do not want to install locally.

You can exit the `devbox` environment with the
```sh
exit
```
command.

Currently there is one issue I encountered with `devbox`, which might be good to know about: environment variables that are set up in more lines, loose their line endings. For example, the environment variable
```sh
export TEST_ENV_VAR="line 1
line 2
line 3"
```
is printed out as
```sh
line 1line 2line 3
```
in the `devbox` shell. This might happen if you have private keys or `k8s` cluster configurations as environment files.