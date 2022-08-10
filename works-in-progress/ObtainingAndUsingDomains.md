## Intro
If you watch youtube you have probably seen countless sponsors for buying a domain.

Up until recently I really didnt feel a need to, but after making my Personal Site, I felt like it would be nice to have a proper domain instead of `https://codef53.github.io/personal-site/`.

While this can be followed like a tutorial, it assumes you already have a site on github pages, and only gives exact directions for domains bought through Google Domains.

## Thinking of a name
I thought of this last night in my sleep, I woke up with the excellent idea of buying `https://f53.dev`. It would be a double win, I have a fun URL and a intresting topic for blog.

I dont recommend this strategy:
- sleeping is an unreliable method for brainstorming
- you should have more reason for buying and making a website than
    - I thought of f53.dev in my sleep
    - it would work well for a blog post

For example, if you arent using github pages to host, you should probably look into if your domain will require a SSL certificate, because not all hosts will provide one.

## Trying to find where to buy it.
![google search of buy domain](https://i.imgur.com/M0PEqhm.png)

I went to the top 6 results that looked like they were actually selling domains, looked up F53.dev, and noted their prices.

| Store | Price |
|---|---|
|Bluehost|unavailable|
|MailChimp|unavailable|
|GoDaddy|$20 (for the first year)|
|Domain.com|$15 + $9 privacy upsell + scare tactics for removing upsell|
|Google|$12|
|NameCheap|$13|

This made Google the clear winner, but it may not be as clear cut for you.

In your own search, some of the other services may look cheaper, but will have a big asterisk of "(for the first year)" meaning they will charge more per year afterwards. 

I would avoid Domain.com because, if they are willing to scare you into an upsell, what other hidden garbage are they going to throw at you. Try avoiding sites with privacy upsells in general, Google provides the same privacy protection features by default, without charging any extra.

## Linking a github pages site to a url you own
### Is it Possible?
Before I buy, I want to check if its even possible to do this, because I dont want to have to also start paying for a server to host stuff.

[Here are the docs on this exact thing](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/about-custom-domains-and-github-pages#using-an-apex-domain-for-your-github-pages-site), they are pretty indepth, so yeah, it looks like this is indeed a thing you can do. They basically immediatly require you work with your DNS Provider*, so there is no testing the waters before you buy your domain.

\* For clarification, DNS Provider here is Domain Name Service Provider, which for me is Google Domains
While checking out f53.dev, I got a warning about .dev domains requiring SSL certificates.\
!["Warning that says \".dev is a secure namespace. You may purchase f53.dev now, but it will require an SSL certificate for website connection\""](https://i.imgur.com/xfZkJV9.png)\
Looking into this, [its brutally simple to set up SSL certificates when using github pages](https://docs.github.com/en/pages/getting-started-with-github-pages/securing-your-github-pages-site-with-https)

### Step 1: setting up the your DNS
Once you have bought your domain, go into the DNS settings in the website you bought your domain from.

Then, create an `A` record that points your domain, for example: `example.com` or `f53.dev` to the following ip addresses:
```
185.199.108.153
185.199.109.153
185.199.110.153
185.199.111.153
```

You will also what to create a `CNAME` record that points your domain, with `www` this time, for example, `www.example.com` or `www.f53.dev` to `<user>.github.io`, for example `codef53.github.io`

If your DNS Provider asks you to enter in your own TTL, it will be how long in seconds a user's pc's will remember where your domain points.\
If you arent doing anything fancy, you should just set this to the biggest value that your domain provider suggests, for example 21600 seconds, or 6 hours.

This is what my finalized entries looked like in google domains:\
![](https://i.imgur.com/IV2meUh.png)

Here is me running the command for my site, displaying the correct output\
![](https://i.imgur.com/h3SmxdK.png)

### Step 2: configuring the github side of things:

Now, go to the source code repository for your github pages site, Go to the settings of your repo:\
![](https://i.imgur.com/eqMqayf.png)

Click into the pages tab:\
![](https://i.imgur.com/VhBpcEn.png)

Scroll down, type in the domain you bought into the "Custom Domain" box, and click save\
![](https://i.imgur.com/dkzRPOL.png)

imgur.com/9AXpm2Q.png)
### Step 3: wait it was that easy?
yeah.\
![](https://i.imgur.com/z3SX7j2.png)

## Custom Email Address
//TODO: CHECK THIS STILL ADDS UP

Now that I own F53.dev it should be pretty simple to set up a custom email like code@f53.dev

Google actually offered me this at checkout of the URL:\
![screenshot of google's custom email options, not really intersting stuff](https://i.imgur.com/f51m5sk.png)

I probably wont do this because [google seemingly treats their premium users worse](https://youtu.be/fiXjR-AhSqs?t=71).