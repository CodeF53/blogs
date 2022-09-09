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

Thinking long and hard about the issue, I came up with a brute force a solution.

> Why don't we just close the Preferences, reopen it, then open the tab the user clicked?

Looking into how to simulate the clicks I needed for this, I came across the [docs for `EventTarget.dispatchEvent()`](https://developer.mozilla.org/en-US/docs/Web/API/EventTarget/dispatchEvent)

As a quick test, I tried to close the preferences modal using this command in Slack's dev tools.

![](https://images2.imgbox.com/a7/2e/vtihfCNS_o.gif)

Given that success, I wrote up the following javascript:

```js
customTab.addEventListener("click", ()=>{
    ...
    // make it so when you click a different tab, it doesn't crash you
    ([...settingsTabList.children]).slice(0,-1).map((tab)=>{
        tab.addEventListener("click", (event)=>{
            // prevent the crash that normally happens
            event.stopPropagation()
            // close the preferences window
            document.querySelector(`[aria-label="Close"]`).dispatchEvent(new Event("click", {bubbles:true}))
            // re-open the preferences window
            document.querySelector(".p-ia__nav__user__button").dispatchEvent(new Event("click", {bubbles:true}))
            document.querySelector("div.ReactModalPortal div:nth-child(7) div").dispatchEvent(new Event("click", {bubbles:true}))
            // go to the tab that was clicked
            tab.dispatchEvent(new Event("click", {bubbles:true}))
        })
    })
}
```

This led to a rather interesting bug. Slack completely froze for around half a second, then the preferences screen was switched to the default "Notifications" tab, with lots of Custom CSS tabs.

![](https://images2.imgbox.com/fb/99/stORn974_o.gif)

I spent 30 minutes trying to figure out why this was happening. Turns out, I was triggering my own code that detects clicking the `Preferences` tab, causing an infinite loop. The freeze was my code recursively triggering itself until it reached a stack overflow.

To fix this, I just added a check to see if the preferences menu click was actually trusted. Real click events have `event.isTrusted` true, while our simulated clicks have `event.isTrusted` false.

```js
document.addEventListener("click", (event) => {
    let element = event.target 
    if (element.classList[0]=="c-menu_item__label" && element.innerHTML == "Preferences" && event.isTrusted) {
        setTimeout(function () {
            if (document.querySelector(".p-prefs_dialog__menu") != null) {
                addSettingsTab()
            }
        }, 50);
    }
})
```
along with that, I added a call to our `addSettingsTab()` so the Custom CSS tab would be there.
 