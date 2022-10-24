## MacBook M1 Pro

Terminal setting

#### Add Color to Text in the zsh Prompt

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


SSH Tunnel
ssh -L 127.0.0.1:8080:10.211.55.5:80 root@10.211.55.5

check port ugage
netstat -anp tcp | grep LISTEN
```

 Move and resize windows in macOS using keyboard shortcuts or snap areas [Rectangle](https://rectangleapp.com/)
