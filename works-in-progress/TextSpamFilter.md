(Blog Title: Making a better spam filter than google)
## Intro
At some point in my life I got on someone's bad side and they did something horrendous, signed my phone number up to as much spam as they could.

Because of this, I now get political spam nearly daily.

<details>
<summary>Here are all the spam texts I have received in the past 2 days:</summary>
![](https://media.discordapp.net/attachments/1009456591914942544/1014380213754404874/Screenshot_20220828-092734.jpg?width=302&height=654)
![](https://media.discordapp.net/attachments/1009456591914942544/1014380214043807754/Screenshot_20220830-214151.jpg?width=529&height=654)
![](https://media.discordapp.net/attachments/1009456591914942544/1014380214312247306/Screenshot_20220830-214207.jpg?width=487&height=655)
![](https://media.discordapp.net/attachments/1009456591914942544/1014380214626816080/Screenshot_20220830-214225.jpg?width=415&height=655)
![](https://media.discordapp.net/attachments/1009456591914942544/1014544359271694466/Screenshot_20220831-083627.jpg?width=449&height=654)
</details>
<small>* don't be an idiot and go to the links in these</small><br>
<small>** don't assume my political opinion. I don't care who it is, I don't want anyone begging for money in my texts</small><br>
<small>*** some of these have "stop2quit". I spent 2 months replying stop to every single one and had no effect. </small>

This would be absolutely fine as long as they were properly filtered out into spam. Problem is, they aren't.

This is absolutely shocking to me for the following reasons:
- I use Google Messages
    - Google is a lead innovator in the neural network space and already have a amazing spam filter for gmail.
- I took a Highschool class in Data Structures and Machine Learning
    - One of the first things we did was spam filtering, and it looked easy.

## The Plan

I am going to make a neural network that predicts if a message is spam or not.

The training data will be the first two messages received from each number in my contacts, along with a boolean marking if they are spam or not:

| Message Content | Is Spam |
| --- | ----------- |
| FedEx: Welcome! Reply YES to opt-in and receive future notifications on packages you want to track. Msg&data rates may apply. Reply HELP for help. | False |
| Senator Michael Bennet here. Nat'l Republicans have started spending their $331m warche...[lots of rambling]...Chip in before our end-of-month deadline tonight? [URL CENSORED] Stop2quit | True |

## Getting the data

Problem with neural networks is they need lots of data. I don't want to spend hours manually getting the first message I have received from every number.

Looking into exporting text data I found a guide saying I should do "Google One"

I downloaded the app and started backing up my phone.

This is taking awhile.

