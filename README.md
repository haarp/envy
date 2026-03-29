## Description

Take your environment with you when use `ssh`, `sudo` or `su` :3

envy's goal is to avoid having to duplicate, deploy and maintain your environment across various user accounts and hosts, by having a single central source-of-truth. It does this by wrapping around the commands `ssh`, `sudo` and `su` and bringing along predefined files. These files become available under a temporary directory exported as the `$ENV_HOME` environment variable on the target.

envy does not require symlinks to work. envy by itself does *not* modify anything inside the target's `$HOME` or leave temporary files around. Multiple people/sessions could be logged into the same user on the same host at the same time without interferring with each other.

envy can be *chained*! If you add envy itself to `envy.d`, it becomes available on the target. Thus you could do `envy ssh foo` -> `envy ssh bar` -> `envy sudo` while keeping the same environment.

## Usage

Prepend `envy` to the desired command. e.g. `envy ssh user@server` or `envy sudo`.

## Example

You add `.bashrc`, `.gitconfig` and `.config/htop/htoprc` to `envy.d`. You run `envy ssh user@host`. envy will transfer these files to the target. On the target a directory `/tmp/env-username.gQGnVn` will be created with contents `.bashrc`, `.gitconfig` and `.config/htop/htoprc`, and exported as `$ENV_HOME`. The shell will launch using that bashrc.

## Requirements

- bash-4.4+ (might be compatible with zsh too, untested)
- tar, xz-utils, base64(coreutils) (on both ends!)

## Configuration

envy is configured through `$XDG_CONFIG_HOME/envy.d/`, which defaults to `$HOME/.config/envy.d/`. The exception to this is if you're already inside an envy session, then the path changes to `$ENV_HOME/.config/envy.d/`

Files underneath here are brought along into the target's `$ENV_HOME`. It is highly recommended to use symlinks where appropiate. Symlinks will be followed. Structure will be kept. A prime candidate to put here is your `.bashrc`. You may want to include `envy` itself to make it available on the target aswell.

The special file `env_commands` if present will be sourced at the end of the target's `.bashrc` when the shell starts. Its main use is to configure applications to use `$ENV_HOME` over `$HOME`. Uses shell syntax and resolves variables on the target accordingly.

Example config is available as `envy.d.example/`. Copy it to `$HOME/.config/envy.d` and rename.

## Terminology

- __master__ is the machine originally executing envy and contains the envy config files and environment to bring.
- __target__ is the user or host the environment will be transferred to. This can be another user on the same machine (usually `root` in the case of `sudo`/`su`) or another host (in the case of `ssh`)
- __environment__ describes the files beneath `envy.d/` we bring. We are not actually taking environment variables with us, if you need them, have your bashrc set them up.

## Caveats (WIP)

- ssh: Cannot be used together with a command, e.g. this won't work: `envy ssh user@server 'ls -l'`.
- ssh: The new shell might be considered a non-login shell, thus skipping displaying the MOTD and such.
- ssh: If your env becomes very large, connections to some ssh servers like `dropbear` might mysteriously fail with errors such as `Broken pipe`. See [here](https://github.com/mkj/dropbear/issues/177) for details.
- sudo|su: If a command is supplied (e.g. `envy sudo nano /etc/hosts`), the new shell considers itself non-interactive. So your bashrc should *not* `return` in this condition. `shopt -s expand_aliases` might also be a worthwhile addition.
- sudo: Will start an interactive session if no command is given, don't use `-i`.
- If you require different configurations for different hosts, have your `.bashrc` handle that.

## bash-completion

To do tab-completion with envy, copy or link `bash-completion` into `~/.local/share/bash-completion/completions/envy`.

## Inspiration

Most of the work started in my [LC_BASHRC](https://github.com/haarp/dotfiles/blob/e5456a112e57114a0bcf075909c471731ae611d6/.bashrc#L13) trick I previously used in my bashrc. By abusing most ssh server's defaults of accepting `LC_*` environment variables, I could take the bashrc with me through that, then source it on the other side with ssh's `RemoteCommand`. sudo and su worked similarly to how they do now, by writing temporary scripts.

Minor credits for the idea of making envy a wrapper go to [sshrc](https://github.com/cdown/sshrc). It attempts to do something similar, although more complex and much less powerful.

envy may look simple, but it went through many iterations and trial&error to arrive at the methods used now.
