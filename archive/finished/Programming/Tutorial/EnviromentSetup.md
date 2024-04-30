## Nice shit
### Good terminal
http://kevinprogramming.com/using-zsh-in-windows-terminal/

## The important shit
This is what ya need

### node
```sh
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.39.0/install.sh | bash
```
restart
```
nvm install --lts
```

### ruby/rvm
```
sudo apt-get install software-properties-common
sudo apt-add-repository -y ppa:rael-gc/rvm
sudo apt-get update
sudo apt-get install rvm
sudo usermod -a -G rvm $USER
```
restart
```
rvm install 2.7.4 --default
gem update --system
```

### git
```
sudo add-apt-repository ppa:git-core/ppa
sudo apt update
sudo apt install git
```

Good config defaults
```
git config --global color.ui true
git config --global init.defaultBranch main
git config --global core. editor "code"
```

ssh shit for github
```
ssh-keygen
```
dont enter passwords

```
cat ~/.ssh/id_rsa.pub | clip.exe
```

Paste into [https://github.com/settings/ssh/new](Github ssh key shit)