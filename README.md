# Instructions to spin up container for development

- 1. Install VS Code Remote containers extension.
- 2. mkdir "your dev dir"
- 3. unzip to the "your dev dir"
- 4. $ mv dot-devcontainer .devcontainer (renames dot-devcontainer to .devcontainer).
- 5. Modify Docker, devcontainer to your liking.
- 6. Open folder .devcontainer in CS Code. It should ask with pop up message to open in container.



# Customize Your Bash

#### Backup the .bashrc file and create necessary files for custom bash.
```
cd
mv .bashrc .bashrc-backup
```

#### Vim to create and modify .bashrc file:
```
vim .bashrc
```

#### Copy and paste the following script into .bashrc file:
```
[ -n "$PS1" ] && source ~/.bash_profile;
```

#### Vim to create and modify .bash_profile file:
```
vim .bash_profile
```

#### Copy and paste the following script into .bash_profile file:
```
# Add `~/bin` to the `$PATH`
export PATH="$HOME/bin:$PATH";

# Load the shell dotfiles, and then some:
# * ~/.path can be used to extend `$PATH`.
# * ~/.extra can be used for other settings you don’t want to commit.
for file in ~/.{path,bash_prompt,exports,aliases,functions,extra}; do
	[ -r "$file" ] && [ -f "$file" ] && source "$file";
done;
unset file;

# Case-insensitive globbing (used in pathname expansion)
shopt -s nocaseglob;

# Append to the Bash history file, rather than overwriting it
shopt -s histappend;

# Autocorrect typos in path names when using `cd`
shopt -s cdspell;

# Enable some Bash 4 features when possible:
# * `autocd`, e.g. `**/qux` will enter `./foo/bar/baz/qux`
# * Recursive globbing, e.g. `echo **/*.txt`
for option in autocd globstar; do
	shopt -s "$option" 2> /dev/null;
done;

# Add tab completion for many Bash commands
if which brew &> /dev/null && [ -r "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]; then
	# Ensure existing Homebrew v1 completions continue to work
	export BASH_COMPLETION_COMPAT_DIR="$(brew --prefix)/etc/bash_completion.d";
	source "$(brew --prefix)/etc/profile.d/bash_completion.sh";
elif [ -f /etc/bash_completion ]; then
	source /etc/bash_completion;
fi;

# Enable tab completion for `g` by marking it as an alias for `git`
if type _git &> /dev/null; then
	complete -o default -o nospace -F _git g;
fi;

# Add tab completion for SSH hostnames based on ~/.ssh/config, ignoring wildcards
[ -e "$HOME/.ssh/config" ] && complete -o "default" -o "nospace" -W "$(grep "^Host" ~/.ssh/config | grep -v "[?*]" | cut -d " " -f2- | tr ' ' '\n')" scp sftp ssh;

# Add tab completion for `defaults read|write NSGlobalDomain`
# You could just use `-g` instead, but I like being explicit
complete -W "NSGlobalDomain" defaults;

# Add `killall` tab completion for common apps
complete -o "nospace" -W "Contacts Calendar Dock Finder Mail Safari iTunes SystemUIServer Terminal Twitter" killall;
```

# Silence ZSH warning.
export BASH_SILENCE_DEPRECATION_WARNING=1

# General Aliases
alias ll='ls -la'
alias cls='clear'
alias py='python3'

#### Vim to create and modify .bash_prompt file:
```
vim .bash_prompt
```

#### Copy and paste the following script into .bash_prompt file:
```
#!/usr/bin/env bash

# Shell prompt based on the Solarized Dark theme.
# Screenshot: http://i.imgur.com/EkEtphC.png
# Heavily inspired by @necolas’s prompt: https://github.com/necolas/dotfiles
# iTerm → Profiles → Text → use 13pt Monaco with 1.1 vertical spacing.

if [[ $COLORTERM = gnome-* && $TERM = xterm ]] && infocmp gnome-256color >/dev/null 2>&1; then
        export TERM='gnome-256color';
elif infocmp xterm-256color >/dev/null 2>&1; then
        export TERM='xterm-256color';
fi;

prompt_git() {
        local s='';
        local branchName='';

        # Check if the current directory is in a Git repository.
        git rev-parse --is-inside-work-tree &>/dev/null || return;

        # Check for what branch we’re on.
        # Get the short symbolic ref. If HEAD isn’t a symbolic ref, get a
        # tracking remote branch or tag. Otherwise, get the
        # short SHA for the latest commit, or give up.
        branchName="$(git symbolic-ref --quiet --short HEAD 2> /dev/null || \
                git describe --all --exact-match HEAD 2> /dev/null || \
                git rev-parse --short HEAD 2> /dev/null || \
                echo '(unknown)')";

        # Early exit for Chromium & Blink repo, as the dirty check takes too long.
        # Thanks, @paulirish!
        # https://github.com/paulirish/dotfiles/blob/dd33151f/.bash_prompt#L110-L123
        repoUrl="$(git config --get remote.origin.url)";
        if grep -q 'chromium/src.git' <<< "${repoUrl}"; then
                s+='*';
        else
                # Check for uncommitted changes in the index.
                if ! $(git diff --quiet --ignore-submodules --cached); then
                        s+='+';
                fi;
                # Check for unstaged changes.
                if ! $(git diff-files --quiet --ignore-submodules --); then
                        s+='!';
                fi;
                # Check for untracked files.
                if [ -n "$(git ls-files --others --exclude-standard)" ]; then
                        s+='?';
                fi;
                # Check for stashed files.
                if $(git rev-parse --verify refs/stash &>/dev/null); then
                        s+='$';
                fi;
        fi;

        [ -n "${s}" ] && s=" [${s}]";

        echo -e "${1}${branchName}${2}${s}";
}

if tput setaf 1 &> /dev/null; then
        tput sgr0; # reset colors
        bold=$(tput bold);
        reset=$(tput sgr0);
        # Solarized colors, taken from http://git.io/solarized-colors.
        black=$(tput setaf 0);
        blue=$(tput setaf 153);
        cyan=$(tput setaf 37);
        green=$(tput setaf 71);
        orange=$(tput setaf 166);
        purple=$(tput setaf 125);
        red=$(tput setaf 124);
        violet=$(tput setaf 61);
        white=$(tput setaf 15);
        yellow=$(tput setaf 228);
else
        bold='';
        reset="\e[0m";
        black="\e[1;30m";
        blue="\e[1;34m";
        cyan="\e[1;36m";
        green="\e[1;32m";
        orange="\e[1;33m";
        purple="\e[1;35m";
        red="\e[1;31m";
        violet="\e[1;35m";
        white="\e[1;37m";
        yellow="\e[1;33m";
fi;

# Highlight the user name when logged in as root.
if [[ "${USER}" == "root" ]]; then
        userStyle="${red}";
else
        userStyle="${orange}";
fi;

# Highlight the hostname when connected via SSH.
if [[ "${SSH_TTY}" ]]; then
        hostStyle="${bold}${red}";
else
        hostStyle="${yellow}";
fi;

# Set the terminal title and prompt.
PS1="\[\033]0;\W\007\]"; # working directory base name
PS1+="\[${bold}\]"; # newline
PS1+="\[${userStyle}\]\u"; # username
PS1+="\[${white}\] at ";
# PS1+="\[${hostStyle}\]\h"; # host
# PS1+="\[${white}\] in ";
PS1+="\[${green}\]\w"; # working directory full path
PS1+="\$(prompt_git \"\[${white}\] on \[${green}\]\" \"\[${yellow}\]\")"; # Git repository details
PS1+="\n";
PS1+="\[${white}\]\$ \[${reset}\]"; # `$` (and reset color)
export PS1;

PS2="\[${yellow}\]→ \[${reset}\]";
export PS2;
```

#### Run the following command to restart the bash:
```
source ~/.profile
```
