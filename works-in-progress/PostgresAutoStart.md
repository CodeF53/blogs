## Intro
PostgresSQL is what basically everyone who uses Ruby on Rails uses.

But it's annoying because you have to manually start it every time you restart WSL.

![image showing the vague annoying error that happens when postgres isn't running](https://i.imgur.com/DhptGOC.png)

This will go over how to auto start it.

## Tutorial
### Why not just .zshrc?
While it's technically possible to auto-start PostgreSQL by adding a line to your `.zshrc` or `.bashrc` file like this:
```
echo '<your_password>' | sudo -S service postgresql start
```
it's not recommended due to security concerns.

So instead, we will be using a more secure approach that allows your user account to run the `service postgresql start` command as root without entering your password.

### Using the Sudoers File
Instead, we'll use a more secure approach that allows your user account to run the `service postgresql start` command as root without entering your password.

Start by running the following command in your terminal:
```
sudo visudo
```
This will open the sudoers file in your default text editor. The sudoers file is a system file that determines which users are allowed to run which commands as root.

Add the following line to the end of the file, replacing `username` with your actual username:
```
# allow starting postgres without sudo
username ALL=(root) NOPASSWD: /usr/sbin/service postgresql start
```
This line gives your user account permission to run the `service postgresql start` command without entering a password.

### Autorun on startup
Using either your `.zshrc` or `.bashrc`, you make postgres autostart with your terminal.

Start by opening your respective file in your favorite text editor:
```
code ~/.zshrc
```
and add the following line to the end of the file
```sh
# autostart postgres
sudo service postgresql start
```