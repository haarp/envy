## Description

Take your environment with you when use `ssh`, `sudo` or `su` :3

envy's goal is to avoid having to duplicate, deploy and maintain your environment across user accounts and hosts, by having a single central source-of-truth. It wraps around `ssh`, `sudo` and `su` and brings along predefined files. These files become available under the temporary directory `$ENV_HOME` on the target.

envy does not create symlinks to work. envy by itself does *not* touch the target's `$HOME` or leave temporary files around. Multiple people/sessions could be logged into the same account on the same host at the same time without interferring with each other.

envy can be *chained*! Thus you could do `envy ssh foo` -> `envy ssh bar` -> `envy sudo` while keeping the same environment.

envy tries to source your bashrc even for non-interactive shells. Yes, you can finally do `envy sudo <that-alias-you-always-wanted-to-use>`!

## Usage

Prepend `envy` to the desired command. e.g. `envy ssh user@server` or `envy sudo`.

## Example

You add `.bashrc`, `.gitconfig` and `.config/htop/htoprc` to `envy.d`. You run `envy ssh user@host`. envy will transfer these files to the target. On the target a directory `/tmp/env-username.gQGnVn` is created with contents `.bashrc`, `.gitconfig` and `.config/htop/htoprc`, and exported as `$ENV_HOME`. The shell launches using that bashrc.

## Installation

Make sure you meet these requirements (should be easy):

- bash-4.4+ (might be compatible with zsh too, untested)
- tar, xz-utils, base64(coreutils) (on both ends!)

Copy or link `envy` to somewhere within your `$PATH`, e.g. `/usr/bin/` or `~/bin/`. Create `~/.config/envy.d/` and start putting symlinks in there.

To enable envy chaining: Symlink `envy` itself into `~/.config/envy.d/bin/envy`.

For tab-completion: Copy or link `bash-completion` into `/usr/share/bash-completion/completions/envy` or `~/.local/share/bash-completion/completions/envy`.

## Configuration

envy is configured through `$XDG_CONFIG_HOME/envy.d/`, which defaults to `~/.config/envy.d/`. Except inside envy sessions, where it becomes `$ENV_HOME/.config/envy.d/`.

Files underneath here are copied into the target's `$ENV_HOME`. Symlinks are recommended where appropiate. Symlinks will be followed. Structure will be kept. A prime candidate to put here is your `.bashrc`. You may also want to include `envy` itself to enable envy chaining.

The special file `env_commands` if present will be sourced on the target after `.bashrc`. Its main use is to configure applications to prefer `$ENV_HOME` over `$HOME`. Uses shell syntax.

Example config is available as `envy.d.example/`. Copy it to `~/.config/envy.d` and rename.

## Terminology

- __master__ is the machine originally executing envy and contains the envy config files and environment to bring.
- __target__ is the user or host the environment will be transferred to. For `sudo`/`su` this is another user account (usually `root`), for `ssh` another host.
- __environment__ describes the files beneath `envy.d/` we bring. We are *not* actually taking environment variables with us. If you need them, have your bashrc set them up.

## Caveats (WIP)

- ssh: Cannot be used together with a command, e.g. this won't work: `envy ssh user@server 'ls -l'`.
- ssh: The new shell might be considered a non-login shell, thus skipping displaying the MOTD and such.
- ssh: If your env becomes very large, connections to some ssh servers like `dropbear` might mysteriously fail with errors such as `Broken pipe`. See [here](https://github.com/mkj/dropbear/issues/177) for details.
- sudo|su: If a command is supplied (e.g. `envy sudo nano /etc/hosts`), the new shell considers itself non-interactive. We use [BASH_ENV](https://www.gnu.org/software/bash/manual/bash.html#index-BASH_005fENV) to have it source the bashrc regardless, so set up your bashrc to *not* `return` in non-interactive shells. `shopt -s expand_aliases` might also be a worthwhile addition.
- sudo: Will start an interactive session if no command is given, don't use `-i`.
- If you require different configurations for different hosts, have your `.bashrc` handle that.

## Inspiration

Most of the work started in my [LC_BASHRC](https://github.com/haarp/dotfiles/blob/e5456a112e57114a0bcf075909c471731ae611d6/.bashrc#L13) trick I previously used. By abusing most ssh server's defaults of accepting `LC_*` environment variables, I could take the bashrc with me, then source it on the other side with ssh's `RemoteCommand`. sudo and su worked similarly to how they do now, by writing temporary scripts.

Minor credits for the idea of making envy a wrapper go to [sshrc](https://github.com/cdown/sshrc). It does something very similar, although more complex and much less powerful.

envy may look simple, but it went through many iterations and trial&error to arrive at the methods used now.
