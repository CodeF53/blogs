## Intro
After [improving the launcher for my Slack Mod](), its almost ready for release! Before that, I would like to do a couple of tweaks and adjustments to make the whole mod feel a bit more polished.

## Editor Animation:
Slack's preferences window is pretty small, which works perfectly for the menus it has by default. Problem is this can feel a little cramped when writing code:
![](https://i.imgur.com/0mwRDBL.png)

At the end of my first SlackMod blog, I wrote a little bit of CSS into the editor to both demonstrate it's use and solve this problem:
```css
/* Increase width and height of Preferences to allow more room for code */
body > div.c-sk-modal_portal > div > div {
    max-width: 100%!important;
    max-height: 100%!important;
    height:100%;
    width:100%;
}
```

Problem is, the other preferences tabs weren't designed to use this space, so it feels very odd.
![](https://i.imgur.com/bO3dUHm.png)

I solved this by hardcoding it so when you click on the Custom CSS tab, the editor expands:

```js
customTab.addEventListener("click", ()=>{
    ...
    // hardcoded styling for CSS Tab:
    // increase width and height of preferences modal for more code room
    ["max-width","width","max-height","height"].forEach(
        style => document.querySelector(`div[aria-label="Preferences"]`).style[style] = "100%")
}
```

This can feel a bit jarring because Preferences screen instantly "snaps" to taking up the whole screen.

Given that, I added a bit of easing to the hardcoded styling:

```js
// hardcoded styling for CSS Tab:
...
// smoothly expand preferences modal
    document.querySelector(`div[aria-label="Preferences"]`).style["transition"] = "500ms ease all"
```

That feels really nice, but then our Ace editor doesn't use all the space we are giving it!

![](https://images2.imgbox.com/fb/59/xAeNI0ll_o.gif)

I thought this would be a pretty simple fix same idea as the other hardcoded stuff
```js
customTab.addEventListener("click", ()=>{
    ...
    const editor = ace.edit('slackMod-editor');
    ...
    // hardcoded styling for CSS Tab:
    ...
    editor.style["height"] = "100%";
}
```

But that doesn't work, as Ace has some internal CSS determining it's size that takes a very high priority. This is set once when the editor is initialized to fit the div its put in. 

Looking at [Ace's docs](https://ace.c9.io/#nav=howto) we can call `editor.resize()` to make the editor expand to fit it's containing div.

Given that we expand the div containing the div for 500 milliseconds, we could just wait 500 ms, then call `editor.resize()`:
```js
setTimeout(()=>{editor.resize()}, 500);
```

But in practice that looks a bit odd as the editor "snaps" to fit the space after the animation is done:
![](https://images2.imgbox.com/13/3c/rkxbPdLt_o.gif)

I am not too proud of my solution for this, but hey, it works.

```js
// make editor resize editor every 5ms for 500ms
// smoothly expanding it with the preferences modal
for (let i = 5; i<=500; i+=5) {
    setTimeout(()=>{editor.resize()}, i);
}
```

Why 5 milliseconds? It is fast enough to look smooth at even 144fps, while also being evenly divisible into 500.

![](https://images2.imgbox.com/54/62/8JkTvLpK_o.gif)

[Hey, thats pretty good!](https://youtu.be/JeimE8Wz6e4)

## Error on switching to other tabs:
I mentioned this briefly in my first blog on SlackMod, when you click the Custom CSS tab, then select a different category, the Preferences Modal crashes.

![](https://images2.imgbox.com/8f/b6/0HhNjOJG_o.gif)

I spent ages trying to fix this before and eventually gave up.

Giving the problem some more thought, I decided to start by preventing the error that happens when you click a tab.

I figured I could just do something like this

```js
customTab.addEventListener("click", ()=>{
    ...
    // make it so when you click a different tab, it doesn't crash you
    ([...settingsTabList.children]).slice(0,-1).map((tab)=>{
        tab.addEventListener("click", (event)=>{
            // prevent the crash that normally happens
            event.stopPropagation()
        })
    })
}
```

But, no, for some reason event.stopPropagation doesn't do what its supposed to. After a lot of thought of other ways to solve the issue, I remembered how I removed event listeners in a lab before to solve this exact problem!

By replacing an element with a clone of itself, you effectively remove all of that element's event listeners:

```js
element.parentElement.replaceChild(element.cloneNode(true), element)
```

Given this, removing all of the click event listeners from the tabs is easy!

First, to iterate through all our tabs, we have to make an iterable array of them. Spreading our tabList's children, then re-wrapping it as an array works for this:
```js
([...settingsTabList.children])
```

Then, we can do our remove event listener trick on every one of these tabs:
```js
customTab.addEventListener("click", ()=>{
    ...
    ([...settingsTabList.children]).map((tab)=>{
        // remove old click event listeners
        tab.parentElement.replaceChild(tab.cloneNode(true), tab)
    })
})
```

Now that clicking the settings tabs doesn't immediately crash the Preferences screen, we can use our own method to select the tab.

The only way I thought of to "switch" tabs was pretty brute force:

> Why don't we just close the Preferences, reopen it, then open the tab the user clicked?

Looking into how to simulate the clicks I needed for this, I came across the [docs for `EventTarget.dispatchEvent()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)

As a quick test, I tried to close the preferences modal using this command in Slack's dev tools.

![](https://images2.imgbox.com/a7/2e/vtihfCNS_o.gif)

Given that success, I made a helper function for clicking elements and tried using it to pull up the correct screen.

```js
const clickNodeBySelector = (selector) => 
    document.querySelector(selector).dispatchEvent(new Event("click", {bubbles:true}))
...
// replace tab click events with our own click event for switching tabs
([...settingsTabList.children]).map((tab)=>{
    // remove old click event listeners
    tab.parentElement.replaceChild(tab.cloneNode(true), tab)
    // add our own click event
    const tabID = tab.id
    document.getElementById(tabID).addEventListener("click", (event)=>{
        if (event.isTrusted) {
            // close the preferences screen
            clickNodeBySelector(`[aria-label="Close"]`)
            // re-open the preferences window
            clickNodeBySelector(".p-ia__nav__user__button")
            clickNodeBySelector("div.ReactModalPortal div:nth-child(7) div")
            // go to the tab that was clicked
            clickNodeBySelector("#"+tabID)
            // add back the custom css tab
            addSettingsTab()
        }
    })
})
```

This almost worked, but it wasn't switching to a tab after entering the preferences menu.

This is because it's clicking the tab on the preferences screen that we are closing.

I fixed this by adding a slight delay to it:
```js
setTimeout(()=>{clickNodeBySelector("#"+tabID)},delay)
```

What I found annoying is that sometimes this would work with a delay of just 5, and other times I need a delay of atleast 50, which starts to get noticeable.

I actually already fixed a similar problem to this with the `addSettingsTab()` function. I check if the elements I need exist, and if they dont, I try again in a bit.

```js
function addSettingsTab() {
    if (document.querySelector(".p-prefs_dialog__menu") !== null) {
        ...
    } else {
        setTimeout(()=>{addSettingsTab()}, 1)
    }
}
``` 

This effectively makes it run once every millisecond until the element it hooks into exists.

Given that we need this exact same solution again, I broke it out into an abstract function:
```js
function tryTillTrue(expression, callback) {
    setTimeout(()=>{ 
        if (expression()) { callback() } 
        else { tryTillTrue(expression,callback)}
    }, 1)
}
```

Heres how it's implemented in `addSettingsTab()`:
```js
function addSettingsTab() {
    tryTillTrue(()=>document.querySelector(".p-prefs_dialog__menu") !== null, ()=>{
        ...
    })
}
```

We still need a little wait before trying to click into the tab because we still have the problem of clicking the one on the window we are closing:
```js
// go to the tab that was clicked
tryTillTrue(()=>document.querySelector("#"+tabID) !== null, 
    ()=>clickNodeBySelector("#"+tabID))
```

With that, it works!

![](https://imgbox.com/VQki9HVh)

But, it does look pretty jarring as the background flashes and the window instantly shrinks. To fix this, I added a bit of styling to make the window ease in:

```js
// go to the tab that was clicked
tryTillTrue(()=>document.querySelector("#"+tabID) !== null,
    ()=>clickNodeBySelector("#"+tabID), 0.025)
// add back the custom css tab
addSettingsTab()

// make it smoothly shrink back to normal window size
setTimeout(()=>{
    ["max-width","width","max-height","height"].forEach(
        style => document.querySelector(`div[aria-label="Preferences"]`).style[style] = "100%")

    setTimeout(()=>{
        document.querySelector(`div[aria-label="Preferences"]`).style["transition"] = "500ms ease all"
        document.querySelector(`div[aria-label="Preferences"]`).style["height"]="700px"
        document.querySelector(`div[aria-label="Preferences"]`).style["width"]="800px"
    },0.025)
},0.025)
```

Yeah... that code looks absolutely disgusting, but it works:

![](https://images2.imgbox.com/59/1c/GecYJbnJ_o.gif)

Ok yea it renders 2 wrong frames, sue me.