# Project Specific Auto-Environment
Using various environments is helpful for managing dependencies separately for different applications or projects, it can result in messiness once you get more than a few. I have recently switched over to using direnv to manage my environment for key "project" directories. 

My current approach utilizes a few different concepts: 
* direnv
* pyenv
* local bash history

## direnv
[Direnv](https://direnv.net/) is shell extension that allows you to manage your project's shell environment within the working directory instead of in the typical user space. This allows you to manage separate settings for the same environment vars depending on where you are. Or, if you have a lot, it can keep things a bit tidier. 

After [installing](https://direnv.net/docs/installation.html), you will need to [hook direnv into your shell](https://direnv.net/docs/hook.html)

I'm using bash, so I was able to get the basics working using:
```bash
eval "$(direnv hook bash)"
```

### direnv-standard library
direnv comes with a [number of integrated tools](https://direnv.net/man/direnv-stdlib.1.html), including support for local pyenv. By adding an extra line or two into your .envrc you can pick up the correct virtual env by simply CDing into your project directory. 

### Useful "addons"
#### path_add <varname> <path> 
This can be helpful for adding in non-project related scripts or tools that might not be part of your standard PATH

#### layout pyenv <version>
This activates the pyenv version of choice. 

## pyenv
[pyenv](https://github.com/pyenv/pyenv?tab=readme-ov-file#simple-python-version-management-pyenv) is one of many python virtual environment managers. The KF/Chop team recommended it, and it integrates nicely with direnv, so I'm in the process of switching over to use it instead of miniconda and venv. 

[Installation](https://github.com/pyenv/pyenv?tab=readme-ov-file#installation) is easy enough: 
```bash
curl -fsSL https://pyenv.run | bash
```

Just don't forget to install the versions of python you intend on using: 

If you want a specific version, you can get a list of versions using the -l flag shown below. I recommend piping that to more/less or grep, since it's a LONG list
```bash
pyenv install -l
```

Once you know exactly which version, you can install it. You can be more or less specific depending on your needs: 

```bash
pyenv install 3.13
```

To integrate the version into direnv, simply add the following to your .envrc file:
```bash
layout pyenv 3.13.12
```
You must use the full version in the layout command. So, if you just specified 3 or 3.13 or whatever, pay attention to which version was listalled. 

If you want the environment local, you can specify that using the basic python layout after the pyenv command: 
```bash
layout pyenv 3.13.12
layout python
```

## Local Bash History
This has been plaguing me for months, since my laptop tends to restart several times a week thanks to HW issues and general Windows fund. Thanks to direnv, I can have bash switch my history file to a local file inside my project directory. This way, it doesn't matter how many times VUMC, or Microsoft or WSL/VS Code decide to restart my kernal or computer, my history will persist just like it did when I did all of my work from a headless linux machine. 

### direnv "plugin" for local history
Create a new file with the following contents: 

```bash ~/.config/direnv/direnvrc
# Custom helper to automate the local history
layout_local_history() {
  export HISTFILE="$PWD/.bash_history_local"
  echo "Project-specific history enabled: $HISTFILE"
}
```

### Changes to .bashrc
The first thing is to create a bash function that will correctly reset the history behavior as required. You can add these lines directly to your .bashrc file, or create a sister rc file that is sourced from the .bashrc itself. 

```bash
# Hook for direnv...same as above
eval "$(direnv hook bash)"
# Force the shell to write to the current HISTFILE before direnv changes it
# and then load the new one after the change.
_update_history() {
    history -a # Append current session history to current HISTFILE
    history -c # Clear current session history
    history -r # read history from the NEW HISTFILE
}

# Ensure this runs whenever the prompt is drawn (after direnv switches envs)
export PROMPT_COMMAND="_update_history; $PROMPT_COMMAND"
```

### Update your project directory's .envrc file
Add the following to your .envrc files wherever you want to switch your history to a local file
```bash
# Local history file persistence
layout_local_history
```
