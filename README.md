## Description

Take your environment with you when use `ssh`, `sudo` or `su` :3

envy's goal is to avoid having to duplicate, deploy and maintain your environment across various user accounts and hosts, by having a single central source-of-truth. It does this by wrapping around the commands `ssh`, `sudo` and `su` and bringing along predefined files. These files become available under a temporary directory exported as the `$ENV_HOME` environment variable on the target.

envy does not require symlinks to work. envy by itself does *not* modify anything inside the target's `$HOME` or leave temporary files around. Multiple people/sessions could be logged into the same user on the same host at the same time without interferring with each other.

envy can be *chained*! If you add envy itself and its configs to `env_files.conf`, it becomes available on the target. Thus you could do `envy ssh foo` -> `envy ssh bar` -> `envy sudo` while keeping the same environment.

## Usage

Prepend `envy` to the desired command. e.g. `envy ssh user@server` or `envy sudo`.

## Example

You add `.gitconfig` and `.config/htop/htoprc` to `env_files.conf`. envy will transfer these files and `.bashrc` to the target. On the target a directory `/tmp/env-username.gQGnVn` will be created with contents `.bashrc`, `.gitconfig` and `.config/htop/htoprc`, and the shell will launch using that bashrc.

## Requirements

- bash-4.4+ (might be compatible with zsh too, untested)
- tar, xz-utils, base64(coreutils) (on both ends!)

## Configuration

Configuration files live under `$XDG_CONFIG_HOME/envy/`, which defaults to `$HOME/.config/envy/`. The exception to this is if you're already inside an envy session, then the path changes to `$ENV_HOME/.config/envy/`

These config files are currently read:

- __env_files.conf__: List of newline-separated files to take with us into `$ENV_HOME`. Files are relative to `$HOME`, do NOT use absolute paths! Paths will be maintained underneath `$ENV_HOME`. Symlinks will be followed and their target used instead. `.bashrc` is always implicitly included.
- __env_commands.conf__: List of newline-separated commands to execute when spawning the new shell. Will be appended to the remote `.bashrc`. Mainly used to configure applications to use `$ENV_HOME` over `$HOME`. Uses shell syntax and resolves variables in the target shell accordingly.

Lines starting with `#` are ignored.

Example config files are available as `env_files.conf.example` and `env_commands.conf.example`.

## Terminology

- __master__ is the machine originally executing envy and which contains the envy config files and environment to be taken along.
- __target__ is the user or host that the environment will be transferred to. This can be another user on the same machine (usually `root` in the case of `sudo`/`su`) or another host (in the case of `ssh`)
- __environment__ describes the `~/.bashrc` along with other dotfiles of the user's choosing. We are not actually taking environment variables with us, if you need them, have your bashrc set them up.


## Caveats (WIP)

- ssh: Cannot be used together with a command, e.g. this won't work: `envy ssh user@server 'ls -l'`.
- ssh: The new shell might be considered a non-login shell, thus skipping displaying the MOTD and such.
- ssh: If your env becomes very large, connections to some ssh servers like `dropbear` might mysteriously fail with errors such as `Broken pipe`. See [here](https://github.com/mkj/dropbear/issues/177) for details.
- sudo|su: If a command is supplied (e.g. `envy sudo nano /etc/hosts`), the new shell considers itself non-interactive. So your bashrc should not `return` in this condition. `shopt -s expand_aliases` might also be a worthwhile addition.
- sudo: Will start an interactive session if no command is suppplied, don't use `-i`.

## bash-completion

To do tab-completion with envy, copy or link `bash-completion` into `~/.local/share/bash-completion/completions/envy`.


## Inspiration

Most of the work started in my [LC_BASHRC](https://github.com/haarp/dotfiles/blob/e5456a112e57114a0bcf075909c471731ae611d6/.bashrc#L13) trick I previously used in my bashrc. By abusing most ssh server's defaults of accepting `LC_*` environment variables, I could take the bashrc with me through that, then source it on the other side with ssh's `RemoteCommand`. sudo and su worked similarly to how they do now, by writing temporary scripts.

Minor credits for the idea of making envy a wrapper go to [sshrc](https://github.com/cdown/sshrc). It attempts to do something similar, although more complex and much less powerful.

envy may look simple, but it went through many iterations and trial&error to arrive at the methods used now.
