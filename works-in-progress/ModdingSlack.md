## Intro
A lot of people don't like how Discord looks/works, so there are many mods that add customizability to it:
- [BetterDiscord](https://betterdiscord.app/)
- [PowerCord](https://powercord.dev/) (now defunct)
- [GooseMod](https://goosemod.com/) (my personal favorite)

Slack is basically a worse Discord but targeted at professionals instead of gamers.

Given that Slack is worse, you would expect there to be more mods for it. Despite this expectation, the results of my search for a Slack mod were underwhelming. There are absolutely no clients that do it for you, and all of the tutorials on how to inject css/js don't work on modern versions of Slack.
- [2019 Medium Article](https://medium.com/l2code/how-to-customize-slack-694a0cd04493)
- [Github on custom CSS last updated 2020](https://github.com/openark/custom-slack-css)
- [A reddit comment from 2019](https://www.reddit.com/r/Slack/comments/cdonno/comment/eu4vqnv/)

The only way I have found so far that works now is [this general purpose electron injector](https://github.com/tintinweb/electron-inject) on github. But this is a far cry from the ease of use of Discord client mods like GooseMod.

## blah blah blah
I wrote a good bit for this section on my desktop, but I didn't push it so I will add it when I get home later.

Some note about that being planned, but for now we are just hardcoded for testing.
```py
from electron_inject import inject

# linux has no extensions for executables
slack_location = "/usr/lib/slack/slack"
inject_location = "/home/f53/Projects/SlackMod/inject.js"

inject(slack_location, devtools=True, timeout=600, scripts=[inject_location])   
```

For me the injector making F12 open devtools didn't work. Fortunately, slack has a built in `/slackdevtools` command

## Adding a Custom CSS section to the Preferences menu

### Initial Plan

To add this new menu I first got the selector for tab list that I would be adding to. Looking at the HTML I determined it's `p-prefs_dialog__menu` class was a good option as it's short and not used elsewhere

![image of the tab list selected](https://i.imgur.com/lRs0VuA.png)

```js
const settingsTabList = document.querySelector(".p-prefs_dialog__menu")
```

From here, I wrote out the code for a basic test

```js
const settingsTabList = document.querySelector(".p-prefs_dialog__menu")
// make a button
customTab = document.createElement("button")
// Set it's label
customTab.innerHTML = `<span>Custom Tab!</span>`
// add the button to the list we selected
settingsTabList.appendChild(customTab)
```

I expected to get a result something like this

![image of some options and then a poorly formatted Custom Tab!](https://i.imgur.com/7t1nGDA.png)

Problem was literally nothing happened. No matter how many comments I added nothing worked.

That code still should work in theory if everything is loaded, so lets put it in a function for later.

```js
function addSettingsTab() {
    const settingsTabList = document.querySelector(".p-prefs_dialog__menu")
    customTab = document.createElement("button")
    customTab.innerHTML = `<span>Custom Tab!</span>`

    settingsTabList.appendChild(customTab)
}
```

### Working with DOM made entirely with JS

Turns out 100% of the HTML slack has is added by javascript. Because of this, getting any DOM related JS to actually work was a massive pain. This took hours to figure out and even longer to figure out how to explain.

To make sure we add the tab when the preferences screen is open. We could do so after the preferences button was clicked.

![a button labeled Preferences with a css selector above it](https://i.imgur.com/KOXsiiA.png)

```js
// const preferencesButton = querySelector(selector)
preferencesButton.addEventListener("click", (event) => {
    addSettingsTab()
})
``` 

But wait, this preferences button is also in a popout menu, we cant select it either!

To make sure the menu that contains this is open, we can make sure the user popout was hovered

![](https://i.imgur.com/BRPqTMh.png)

```js
// const userPopout = querySelector(selector)
userPopout.addEventListener("hover", (event) => {
    // const preferencesButton = querySelector(selector)
    preferencesButton.addEventListener("click", (event) => {
        addSettingsTab()
    })
})
```

But wait, in some screens the user popout isn't shown!

You may be detecting a pattern here, we cant have any direct selectors to elements unless we know we are in a context that they exist.

There are probably hundreds of solutions to this, but here is the one I came up with.

```js
// cant safely select any HTML, so select root
document.addEventListener("click", (event) => {
    // check if clicked element is the button that opens our screen
    let element = event.target 
    if (element.classList[0]=="c-menu_item__label" && element.innerHTML == "Preferences") {
        addSettingsTab()
    }
})
```

That almost works, but there is one last technicality. Our injected code is runs before Slack's code.

Here is the effective order of things in pseudocode

```
injected code:
    document.click {
        clicked == preferencesButton {
            preferencesScreen.append(button)
        }
    })

slack's code:
    preferencesButton.click {
        make preferencesScreen
    })
```

We try adding a button to the screen before its made!

We can fix this by adding an asynchronous wait command, making our button adding code run after the screen is made.

```js
// setTimeout(function to run, how long from now to run it in milliseconds)
setTimeout(function () { 
    if (document.querySelector(".p-prefs_dialog__menu") != null) {
        addSettingsTab()
    }
}, 500);
```

Here is the "pseudocode" for that, the left pane is thread 1, the right pane is thread 2

![oh god yeah no I am not explaining this](https://i.imgur.com/DnxblpB.png)

In summary for all of this, check if screen will be opened, wait so screen can open, edit screen. 

Heres all that code in full

```js
// cant safely select any HTML, so select root
document.addEventListener("click", (event) => {
    // check if clicked element is the button that opens our screen
    let element = event.target 
    if (element.classList[0]=="c-menu_item__label" && element.innerHTML == "Preferences") {
        // Our injected code is runs before Slack's code.
        // So the screen hasn't been made yet
        // Wait a short bit asynchronously so it can be made
        // Then add it.
        setTimeout(function () {
            if (document.querySelector(".p-prefs_dialog__menu") != null) {
                addSettingsTab()
            }
        }, 500);
    }
})
```

### Adding text editor into the tab

// REVIEW ALL TEXT FROM HERE ON

```js
function addSettingsTab() {
    const settingsTabList = document.querySelector(".p-prefs_dialog__menu")
    customTab = document.createElement("button")
    customTab.innerHTML = `<span>Custom Tab!</span>`
    // onClick
    //     clear pane to the right
    //     add some kind of multiline text form

    settingsTabList.appendChild(customTab)
}
```

looking at "pane to the right" .p-prefs_dialog__panel:

![the right pane in the pref menu](https://i.imgur.com/XB089Dx.png)

multiline text form

`<textarea>`


### Making the text persistent

### Actually using that as CSS

### Syntax Highlighting