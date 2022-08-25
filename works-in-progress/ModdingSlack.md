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

## Injecting JavaScript through Electron Inject
Instead of reinventing the wheel, I decided to make my project depend on the well maintained [electron-inject](https://github.com/tintinweb/electron-inject) github repo.

Here's the script I threw together

```py
from electron_inject import inject

# for windows this will be something like "C:/ProgramData/F53/slack/app-4.27.154/slack.exe"
slack_location = "/usr/lib/slack/slack"
inject_location = "/home/f53/Projects/SlackMod/inject.js"

inject(slack_location, devtools=True, timeout=600, scripts=[inject_location])
```

Developing with this is ~~a pain~~ super easy:
- Make changes to your javascript file
- tab to your slack and alt f4 it
    - make sure you have slack configured so it doesn't go run in the background   
- alt tab to your console, press up and enter to re-run the python script

For me the injector making F12 open devtools didn't work. Fortunately, slack has a built in `/slackdevtools` command

This was supposed to be a temporary solution for me while I made this mod. With the deadline on this blog coming up, I will make my own automatic injector sometime later.

## Adding a Custom CSS section to the Preferences menu

### Goal:
Eventually, I want the custom tab to be similar to Topaz's snippets section, where there is essentially a file picker editing/enabling/disabling individual css files
![Topaz Snippets](https://i.imgur.com/aVWk3pG.png)

But currently, I have no idea how to save a file, so the goal for today is something more like Goosemod's Custom CSS, which has one editor.
![Goosemod Custom CSS Screen](https://i.imgur.com/gOUwBp1.png)

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

Heres the plan:
```js
function addSettingsTab() {
    const settingsTabList = document.querySelector(".p-prefs_dialog__menu")
    customTab = document.createElement("button")
    customTab.innerHTML = `<span>Custom CSS</span>`
    // add class that make look good

    // onClick
    //     deselect old tab
    //     select new tab
    //     clear pane to the right
    //     add some kind of multiline text form to the pane

    settingsTabList.appendChild(customTab)
}
```

Add class that make look good:
- copy all classes from one of the other buttons
- set the custom tab to have those

```js
// add class that make look good
customTab.classList = "c-button-unstyled c-tabs__tab js-tab c-tabs__tab--full_width"
```

I honestly didnt expect it to be that easy and thats why I made it it's own step.

![the normal slack preferences tabs but with custom css at the bottom looking hot](https://i.imgur.com/JrCh9pp.png)

Tabs being visually selected is dependent on if they have one class `c-tabs__tab--active`. So doing the selection stuff is as this:

```js
const activeClass = "c-tabs__tab--active"
// get old tab
let activeTab = settingsTabList.querySelector("."+activeClass)
// visually deselect old tab by removing class
activeTab.classList = // classList but without activeClass
// visually select new tab by adding class
customTab.classList = customTab.classList.toString() + " " + activeClass
```

Removing the selected class at first seems pretty troubling because by default activeTab.classList is an array of some custom object, but calling `activeTab.classList.toString()` lets us just use `.replace(stringToRemove, "")`

That gives us the following:

```js
// visually deselect old tab by removing class
activeTab.classList = activeTab.classList.toString().replace(activeClass+" ", "")
// visually select new tab by adding class
customTab.classList = customTab.classList.toString() + " " + activeClass
```

That looks pretty good!
![prior image but custom css is now selected](https://i.imgur.com/RhyRDa9.png)

Making a text editor is super simple
```js
// make the element
let cssEditor = document.createElement("textarea")
// size it 
cssEditor.setAttribute("rows", "33")
cssEditor.setAttribute("cols", "60")
```

The rest of our current plan can be done in 1 line. Basically we just throw out whatever the old Preferences tab was showing and put in our textarea

```js
// clear pane to the right
// add some kind of multiline text form to the pane
document.querySelector(".p-prefs_dialog__panel").replaceChildren(cssEditor)
```

Now we have a text editor!

![the slack preferences page, but with custom css selected and a empty text window](https://i.imgur.com/x6R8Zpm.png)

### Actually using input as CSS
[This stackoverflow answer](https://stackoverflow.com/a/707580) is great for arbitrary CSS not specific to a node. The following is loosely based on it for our purposes:

```js
// make a new style element for our custom CSS
let styleSheet = document.createElement("style")
// set default contents of Custom CSS
styleSheet.innerText = "/*Write Custom CSS here!*/"
// give it an id to make it easier to query
// the document for this stylesheet later
styleSheet.id = "SlackMod-Custom-CSS"
// add to head
document.head.appendChild(styleSheet)
```

We cant just do `styleSheet.innerText = new value` because styleSheet is a static reference. Instead we query the document for the ID we gave it and then set the css from there: 

`document.querySelector("#SlackMod-Custom-CSS").innerText = newCSS;`

To make this cleaner, I made 2 methods

```js
// method to quickly change css
const updateCustomCSS = newCSS => { document.querySelector("#SlackMod-Custom-CSS").innerText = newCSS; }
// method to quickly get inner css
const getCustomCSS = () => { return document.querySelector("#SlackMod-Custom-CSS").innerText}
```

Now we just need to make our `cssEditor` run these accordingly
```js
// a big proper editor
let cssEditor = document.createElement("textarea")
cssEditor.setAttribute("rows", "33")
cssEditor.setAttribute("cols", "60")
// set content from current CSS
// on new chars added
    // update current CSS
```

Setting content is easy
```js
cssEditor.setAttribute("rows", "33")
cssEditor.setAttribute("cols", "60")
// set content from current CSS
cssEditor.value = getCustomCSS()
```

Finding which event listener to use for when new characters are added to this editor was more difficult, but by reading through the [list of all event listeners](https://developer.mozilla.org/en-US/docs/Web/Events#event_listing) I found what I needed, the horribly named ["input" event](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/input_event).

From it's description, its exactly what we need:
> The input event fires when the value of an `<input>`, `<select>`, or `<textarea>` element has been changed.

But it's name is `input`, not `inputChanged` or something descriptive, just `input`

Once we do know the horribly bad name for the event listener we need, making the css update in real time is easy:
```js
// on new chars added
cssEditor.addEventListener("input", ()=>{
    // update current CSS
    updateCustomCSS(cssEditor.value)
})
```

With that, now we are moving!
![a screenshot of the preferences panel up, set to custom css, with some custom css making the background of the tabs red, and the background of the editor a darker gray](https://i.imgur.com/RsFI1VH.png)

Now we can write css and see it update as we type!

### Making CSS editor behave and look right
There are a few things preventing this custom CSS panel from feeling usable though.
- tab kicks you out of the box instead of indenting you
- the font isn't monospace
- upon exiting and re-entering the screen all your newlines are gone
    - no idea why this happens or how to fix it

Fixing tabs kicking you out is easy, simply prevent default behavior of the tab key when you are in the css editor.
```js
// make pressing tab add indent
cssEditor.addEventListener("keydown", (event) => {
    if (event.code == "Tab") {
        event.preventDefault();
    }
})
```
I have no idea how to make it actually indent you though.

To fix the monospace font issue, I just changed the default value of our CSS field to be this instead of just `/*Write Custom CSS here!*/`
```css
/*Write Custom CSS here!*/
.p-prefs_dialog__panel textarea {
   font-family: Monaco,Menlo,Consolas,Courier New,monospace!important;
}
```

## Current Limitations
Currently, this mod is limited in several ways:
- CSS does not persist between restarts
    - looking into Javascript file I/O, [there is apparently no direct way to save a file to a user's system](https://stackoverflow.com/questions/21012580/is-it-possible-to-write-data-to-file-using-only-javascript)
- Injecting is manual and hardcoded to the user's system
    - Every time I have changed the javascript I am injecting throughout writing this blog I have
        - alt+f4'd slack
        - tabbed to my terminal
        - pressed up and enter to rerun `python slack_launch.py`
        - waited ~30 seconds 
- Selecting custom css in preferences then selecting a different category leads to an issue
- No syntax highlighting

Normally I would delay release of the blog until all of these issues were fixed, but I have a deadline for releasing this one.

If you want to help solving these issues all the code discussed can be found at [github.com/CodeF53/SlackMod](https://github.com/CodeF53/SlackMod). Otherwise, stay tuned for a followup where I fix these issues.