## Intro
A lot of people dont like how Discord looks/works, so there are many mods that add customizability to it:
- [BetterDiscord](https://betterdiscord.app/)
- [PowerCord](https://powercord.dev/) (now defunct)
- [GooseMod](https://goosemod.com/) (my personal favorite)

Slack is basically a worse Discord but targed at professionals instead of gamers.

Given that Slack is worse, you would expect there to be more mods for it. Despite this expectation, the results of my search for a Slack mod were underwhelming. There are absolutely no clients that do it for you, and all of the tutorials on how to inject css/js dont work on modern versions of Slack.
- [2019 Medium Article](https://medium.com/l2code/how-to-customize-slack-694a0cd04493)
- [Github on custom CSS last updated 2020](https://github.com/openark/custom-slack-css)
- [A reddit comment from 2019](https://www.reddit.com/r/Slack/comments/cdonno/comment/eu4vqnv/)

The only way I have found so far that works now is [this general purpouse electron injector](https://github.com/tintinweb/electron-inject) on github. But this is a far cry from the ease of use of Discord client mods like GooseMod.

## Making my own client mod for Slack
Given all of this, I set out to make my own Slack mod.