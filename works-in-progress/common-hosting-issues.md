## Info
This is just an archive of all the issues my friends have ran into while following the rails server guide:

## Request Timed Out
![](https://i.imgur.com/1NLmDMB.png)

Start by [navigating to Compute inside your project](https://console.cloud.google.com/compute), and click the link to go into controls for your instance:

![](https://cdn.discordapp.com/attachments/1019985929642979348/1044297103133913158/image.png)

Click edit:

![](https://i.imgur.com/z6X8SIh.png)

Scroll down to `Network interfaces`, under firewalls, enable HTTP and HTTPS traffic.

![](https://i.imgur.com/1NLmDMB.png)

Save

![](https://i.imgur.com/6lYHaJk.png)

## Connection Not Established
![](https://cdn.discordapp.com/attachments/1019985929642979348/1044316894968172705/Screen_Shot_2022-11-21_at_11.22.11_AM.png)

You just gotta start your Postgres Server [run everything except step 3 in this guide](https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-ruby-on-rails-application-on-ubuntu-18-04)

If you get an error like
```
~/proj$ source ~/.bashrc
Please see `nvm --help` or https://github.com/nvm-sh/nvm#nvmrc for more information.
when running `source ~/.bashrc`
```
Just ignore it.