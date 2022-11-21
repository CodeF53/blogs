## Intro:
When I wanted to deploy my server, I was recommended Heroku and Render.

Using those was a pain in the ass because their interfaces are painfully slow and I didn't want to figure out their stupid alternative to DockerFiles.

I knew I could just get a server I can SSH into, but I couldn't find any tutorials online for doing that with Rails.

I find this process much easier than anything else on the market.

## Assumptions:
I assume you already have a project that:
- Are using react on rails
- You can run and access through localhost:3000
- The site files are at, or compiled to `/public`
- Your database is based on PostgreSQL
  - I wrote about migrating to it [in a prior blog](https://dev.to/f53/using-action-cable-with-react-and-rails-2o9f#Switching-an-existing-rails-app-to-postgresql).
  - Migrating to postgres isn't exactly required, but using SQLite in production isn't recommended.

If you don't already have a project that meets these guidelines, and want something to test with, I recommend my [React + Rails Template](https://github.com/CodeF53/react_rails_template).\
<small> This template is based on SQLite, not Postgres </small>

## Google Cloud Setup:
### Getting into Google Cloud
1. [Install the google cloud sdk in your preferred terminal](https://cloud.google.com/sdk/docs/install-sdk)
2. [Create a google cloud project](https://cloud.google.com/resource-manager/docs/creating-managing-projects)
3. [Enable billing on that project](https://cloud.google.com/billing/docs/how-to/verify-billing-enabled)
4. [Enable cloud compute in your project](https://console.cloud.google.com/compute/instances)

### Creating a new SSH instance:
1. [Navigate to Compute inside your project](https://console.cloud.google.com/compute)
2. Click Create Instance:
  ![](https://i.imgur.com/1YZfc1v.png)
3. Select a region for your server to run on
  - [You can find which one will have the best ping for your area using this site](https://gcping.com/)
4. Select the machine you will be running your server on.
  - I recommend starting with E2-micro and moving up if you need more power
5. Enable both HTTP and HTTPS traffic
  ![](https://i.imgur.com/M1dfB8c.png)

### SSHing into your new machine:
[Navigate to Compute inside your project](https://console.cloud.google.com/compute)

Click the dropdown to the right of ssh, click view gcloud command

![](https://imgur.com/4Ag8EgN.png)

Click copy to clipboard and paste that into your terminal

## Setting Up Your Machine:
Google machines run stock Debian 11, without anything pre-installed.

Because of this, once you are SSHed in, first thing you want to do is install the essentials, git, ruby, and node. [I have a blog with all the commands to do this on ubuntu](https://dev.to/f53/environment-setup-stuff-2p1e).

The method I shared there for installing RVM wont work on Debian, [so here is a guide for that](https://rvm.io/rvm/install)

Also install postgres by following [everything except step 3 of this guide](https://www.digitalocean.com/community/tutorials/how-to-use-postgresql-with-your-ruby-on-rails-application-on-ubuntu-18-04)

## Setting up your Rails Server:
Start by cloning down your project and cd into it
```
git clone https://github.com/YOU/project
cd project
```

Install your project dependencies:
```
bundle install

cd client
npm i
cd ..
```

Compile your frontend to `/public`
```
bin/build.sh
```

## Nginx
Nginx does some magic bullshit that makes the server easily accessible, and makes setting up HTTPS much easier later on.

### Setting up Nginx
Inside your Rails project, run `pwd` to get `[PATH TO YOU RAILS PROJECT]/`, which you will use later

Install Nginx
`sudo apt install nginx`

Navigate to the Nginx's site config
`cd /etc/nginx/sites-enabled`

Open a text editor with super user permissions on the file default. Nano is the most user friendly for this
`sudo nano default`

Delete everything that already exists in the file using nano

Paste this in to replace what you just deleted, filling in `[PATH TO YOU RAILS PROJECT]/` with the thing you got from `pwd` earlier
```
upstream rails_server {
  server 0.0.0.0:3000;
}

server {
  listen 80;

  location / {
    root [PATH TO YOU RAILS PROJECT]/public;
    try_files $uri @missing;
  }

  location @missing {
    proxy_pass http://rails_server;
    proxy_set_header Host $http_host;
    proxy_redirect off;
  }
}
```

<kbd>ctrl</kbd>+<kbd>x</kbd> to exit, <kbd>y</kbd> to save, <kbd>enter</kbd> to confirm file name

### Testing your Nginx install
First, restart your server to finish setting up Nginx:

[Navigate back to Compute inside your project](https://console.cloud.google.com/compute), and click the link to go into controls for your instance:

![](https://cdn.discordapp.com/attachments/1019985929642979348/1044297103133913158/image.png)

Click `STOP` on the top bar, wait ~1 minute for `START/RESUME` to become clickable, and click it.

![](https://media.discordapp.net/attachments/1019985929642979348/1044297197145038878/image.png)

Then, SSH back into your server.

Run this to make sure your server is running:
```
sudo systemctl status nginx
```
You should see something like this:
![](https://i.imgur.com/LS52ONI.png)

To get the IP your server is on run this:
```
wget http://ipinfo.io/ip -qO -
```
<small>you may have to `sudo apt install wget`</small>

Then, go to that ip in your browser, you should see something like this:
![](https://i.imgur.com/VadxAQs.png)

## Testing your Rails setup:
Now that we can access our server, we can run rails and use our website:
```
rails s -e production
```

If you refresh the page you were at earlier, you should now be able to see your app!

If you are having issues, click [here for a guide covering some common issues]()

## Automatic Starting and Static IP
### Static IP

### Automatic Starting

## Domain name

## HTTPS
