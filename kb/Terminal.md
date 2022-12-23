# Customizing the Terminal
A config guide for a nice terminal setup.  
December 2022 - Daan Gommers

## Installing zsh
```bash
$ sudo apt install zsh
```
Start `zsh`, and create the `.zshrc` file with just a comment

## Installing antigen
```bash
$ curl -L git.io/antigen > antigen.zsh
$ mv antigen.zsh .config/antigen.zsh
```
Install the fonts using the **manual** method according to the instructions [here](https://github.com/romkatv/powerlevel10k#fonts).

# Configure .zshrc
```bash
nano .zshrc
```
Set content to:
```bash
source ~/.config/antigen.zsh

# Load the oh-my-zsh's library.
antigen use oh-my-zsh

# Bundles from the default repo (robbyrussell's oh-my-zsh).
antigen bundle docker
antigen bundle git
antigen bundle pip
antigen bundle lein
antigen bundle command-not-found

# Syntax highlighting bundle.
antigen bundle zsh-users/zsh-syntax-highlighting
antigen bundle zsh-users/zsh-autosuggestions
antigen bundle zsh-users/zsh-completions

# Load the theme.
antigen theme romkatv/powerlevel10k

# Tell Antigen that you're done.
antigen apply
```
# Add conda to start of line in terminal
Make the following edits to  `~/.p10k.zsh`:

Add `anaconda` between lines 36 and 37:
```
 33  typeset -g POWERLEVEL9K_LEFT_PROMPT_ELEMENTS=(
 34    os_icon                 # os identifier
 35    dir                     # current directory
 36    vcs                     # git status
 37    anaconda                # conda environment
 38    # prompt_char           # prompt symbol
 39  )
```

Comment `anaconda` out on line 52:
```
 45  typeset -g POWERLEVEL9K_RIGHT_PROMPT_ELEMENTS=(
       ...
 50    asdf                    # asdf version manager (https://github.com/asdf-vm/asdf)
 51    virtualenv              # python virtual environment (https://docs.python.org/3/library/venv.html)
 52    # anaconda              # conda environment (https://conda.io/)
 53    pyenv                   # python environment (https://github.com/pyenv/pyenv)
 54    goenv                   # go environment (https://github.com/syndbg/goenv)
       ...
111  )
```

Change color of anaconda on line 965:
```
963 # Anaconda environment color.
964 typeset -g POWERLEVEL9K_ANACONDA_FOREGROUND=0
965 typeset -g POWERLEVEL9K_ANACONDA_BACKGROUND=5 #prev: 4
```
# Change the default shell to zsh
```bash
$ which zsh
/usr/bin/zsh
$ chsh
Password: 
Changing the login shell for daan
Enter the new value, or press ENTER for the default
	Login Shell [/bin/bash]: /usr/bin/zsh
```
Log off and on again in ubuntu
