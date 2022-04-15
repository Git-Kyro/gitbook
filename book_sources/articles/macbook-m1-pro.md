## MacBook M1 Pro

Terminal setting

### Add Color to Text in the zsh Prompt

```sh
[kyro@Kyros-MacBook-Pro ~]#cat ~/.zshrc 

export PROMPT='%B%F{51}[%n@%m %1~]#%f%b'
# PS1="%n@%m %1~ %# "

export CLICOLOR=1
export LSCOLORS=ExGxFxdaCxDaDahbadeche
alias ll='ls -Gl'
alias wget='/opt/homebrew/Cellar/wget/1.21.3/bin/wget'
alias brew='/opt/homebrew/bin/brew'
alias tree='/opt/homebrew/Cellar/tree/2.0.2/bin/tree'
```

### Move and resize windows in macOS using keyboard shortcuts or snap areas

[Rectangle](https://rectangleapp.com/)