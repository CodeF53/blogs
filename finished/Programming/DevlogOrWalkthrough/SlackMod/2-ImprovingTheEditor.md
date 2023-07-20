## Intro
In [my previous blog](https://dev.to/f53/adding-a-custom-css-menu-to-slack-1090) I made a working live css editor for slack inspired by discord mods. I ended that blog with the following list of limitations the mod has:

> - CSS does not save newlines
> - CSS does not persist between restarts
> - Injecting is manual and hardcoded to the user's system
> - Selecting custom css in preferences then selecting a different category leads to an issue
> - No syntax highlighting

This blog will focus on addressing the following of those limitations:
- CSS does not save newlines
- CSS does not persist between restarts
- No syntax highlighting

## How do other mods do it?
I had a idea in a dream last night that looking at how Discord mods add custom CSS could be helpful.

Instead of spending ages problem solving with trial and error to make our own editor we can base our work on the work of other people who have already solved these issues.

### Going over GooseMod's CustomCSS Code

I decided to dig through the [code for GooseMod's css editor](https://github.com/GooseMod-Modules/CustomCSS/blob/main/index.js) for inspiration and ideas.

In the first few lines we can start drawing similarities between our code and theirs:

`main/index.js lines 8-12`
```js
const updateCSS = (c) => {
  styleEl.innerHTML = '';
  
  styleEl.appendChild(document.createTextNode(c));
};
```

This is equivalent to our `updateCurrentCSS` method:
```js
const updateCustomCSS = newCSS => { 
    document.querySelector("#SlackMod-Custom-CSS").innerText = newCSS; 
}
```

Looking at what they did differently, I wondered if fixing the problem of CSS not saving newlines was as simple as changing our method from reassinging `innerText` to appending text nodes instead.
To test this idea, I revised my methods to this:
```js
// method to quickly change css
const updateCustomCSS = newCSS => { 
    document.querySelector("#SlackMod-Custom-CSS").innerHTML = "" 
    document.querySelector("#SlackMod-Custom-CSS").appendChild(document.createTextNode(newCSS); 
}
```
I also changed my get method to use innerHTML instead of innerText:
```js
// method to quickly get inner css
const getCustomCSS = () => { 
    return document.querySelector("#SlackMod-Custom-CSS").innerHTML
}
```

Shockingly, this worked! My text editor actually kept newlines! I would've never thought to try that.

Looking at the remaining code it looks like they use a library called "ace" to create a nice looking text editor

First they import the libraries they will use:

`main/index.js lines 22-27`
```js
// Setup ace
eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js`)).text()); // Load Ace main

eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-monokai.js`)).text()); // Load Monokai theme

eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js`)).text()); // Load CSS lang
```

Then they call a function from the goosemod API to add a settings category, (something I don't have the luxury of doing):

`main/index.js lines 29-34` (edited for clarity)
```js
goosemodScope.settings.createItem('Custom CSS', [
    `(v${version})`, { type: 'custom', element: () => {
        // code that makes and returns the node they 
        // want to be in the settings menu
```     

Inside that, they initialize the div that will contain their editor with some css:

`main/index.js lines 35-43` (edited for clarity and briefness)
```js
// add div for ace to be in
const el = document.createElement('div');
// give it an id so ace can see it
el.id = 'gm-editor';
// size it to user's window
el.style.width = '90%';
el.style.height = '85vh';
// set its default value
el.innerHTML = css;
```

Then, they do 2 things:
- start a new thread that waits 10ms, before initializing Ace
- return the div

This is presumably because they cant initialize the editor until the div that it 
goes in exists.

`main/index.js lines 45-63` (edited for clarity and briefness)
```js
// instantly ran async function (async function() {..})();
(async function() {
    // wait 10 milliseconds (they wrote a method for this earlier)
    await sleep(10);

    // point ace to the id we gave our div
    const editor = ace.edit('gm-editor');
    const session = editor.getSession();
    // configure ace's ????, syntax highlighting, theme
    session.setUseWorker(false); // Tell Ace not to use Workers
    session.setMode('ace/mode/css'); // Set lang to CSS
    editor.setTheme('ace/theme/monokai'); // Set theme to Monokai
    // equivalent to addEventListener("input")
    session.on('change', () => {
        // some magic with localstorage we will get into later
        const val = session.getValue();
        css = val;
        updateCSS(val);
    });
})();

return el;
```

We wont need this async function because we can just add the div to the document before initializing the editor.

## Implementing what we learned
From what I read, it seems like initializing Ace will be easy once we get it imported.

### Importing Ace
Attempting to do exactly what GooseMod did would be nice, but unfortunately it seems Slack has blocked `eval()` on untrusted text. 

Attempting to run in the console:
```js
eval(await (await fetch(`https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js`)).text()); // Load Ace main
```
gives us this error:

```
Uncaught EvalError: Refused to evaluate a string as JavaScript because 'unsafe-eval' is not an allowed source of script
```

I next thought to try and append a `<script>` to the header:

```js
var script = document.createElement('script');
script.type = 'text/javascript';
script.src = "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js";
document.head.appendChild(script);
```

Attempting to run this gives us a much clearer, descriptive error:

![Refused to load the script 'link to script' because it violates the following Content Security Policy directive: "script-src 'self' 'wasm-eval' 'wasm-unsafe-eval' https://*.sdkassets.chime.aws https://*.slack.quip.systems https://quip-cdn.com https://slack-prod.qvpc-cdn.com https://a.slack-edge.com/ https://b.slack-edge.com/](https://i.imgur.com/Jy3AH86.png)

What I think this means is that scripts are only allowed from URLs that match one of the following:
- `https://*.sdkassets.chime.aws`
- `https://*.slack.quip.systems`
- `https://quip-cdn.com`
- `https://slack-prod.qvpc-cdn.com`
- `https://a.slack-edge.com/`
- `https://b.slack-edge.com/`

Then, I thought I could just add the URL for our libraries to the `scripts=[]` argument of our python injecting code:

```py
inject(slack_location, devtools=True, timeout=600, scripts=[
    # libraries
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-monokai.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js",

    # our injected code
    inject_location
]) 
```
But no, the Electron Inject python library expects only actual files to be passed into the scripts.

![FileNotFoundError: [Errno 2] No such file or directory: 'https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js'
](https://i.imgur.com/4LsiVHi.png)

### Improving our inject script

I spent some time adding dynamic library downloading to our launch script.

Here was the basic idea in pseudocode:

```
for each library file we need:
    check if its already downloaded to a libs folder
    if not
        download it
        move it to the libs folder
    add path to lib to the scripts argument 
```

Im not going to go over this line by line but python reads pretty well and this is well commented.

```py
# scripts we will inject are added to this array
scripts = []
# get the directory this python code is in
# or the Current Working Directory
cwd = getcwd()
# make the libs folder if it doesn't exist
libsFolder = f"{cwd}/libs"
if not exists(libsFolder):
    makedirs(libsFolder)
# array of libraries to install
libURLs = [
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/theme-dracula.js",
    "https://ajaxorg.github.io/ace-builds/src-min-noconflict/mode-css.js"]
for libURL in libURLs:
    # gets the last substring that follows a /
    fileName = libURL.split("/")[-1]
    print(f"\nGetting library file: {fileName}")

    # check if we have already downloaded it
    if exists(f"libs/{fileName}"):
        print(f"\tfound {fileName} in libs")
    else:
        print(f"\t{fileName} not in libs, downloading")
        # download it
        download(libURL)
        # move it into libs
        rename(f"{cwd}/{fileName}", f"{libsFolder}/{fileName}")
    # add path to file to list of scripts to inject
    # we use realpath() here to get the fill directory to the file
    # because sometimes electron inject gets screwy with relative files
    scripts.append(f"{libsFolder}/{fileName}")
# add our own code as the last entry in scripts
# so all the scripts are loaded once our code runs
scripts.append(f"{cwd}/inject.js")
```

### Making our editor with Ace
While we couldn't use GooseMod's code directly for imports, its code for setting up ace can practically be copy pasted in place of our old textarea code.

Given we that are using that part of their code in full, I have to include a license. 

I didn't like how much space the MIT license took in my code, so I asked the developer who wrote it if I really needed to put it there:

![Ducko — looks cool, reminds me of a thing i did ages ago just put link to original source as a comment probably | CodeF53 — alright, thanks! | Ducko — and/or include original mit license, up to you it's kinda needed but also not so shrug](https://i.imgur.com/ovY78Kh.png)

Based on their response, I decided to include this comment:

```js
/*
Based on GooseMod CustomCSS, which is under the MIT License
https://github.com/GooseMod-Modules/CustomCSS/blob/64969856598a2cc2980988046e0ff266d64fa943/index.js#L35-L65
*/
```

So what was changed other than variable/method names?

First, I changed the width/height css to the css I wrote for the old editor:


```js
cssEditor.style.width = "100%";
cssEditor.style.height = "calc(100% - 0.5rem)";
// set its default value
cssEditor.innerHTML = getCustomCSS();
```

I used old code I had for adding the editor to the settings content pane.

```js
// add editor to settings content pane
document.querySelector(".p-prefs_dialog__panel").replaceChildren(cssEditor)
```

Because we are adding the editor to the dom instantly, we don't need all the async function waiting stuff.

I also changed the theme set here from Monokai to Dracula, as it looks better.

```js
// point ace to the id we gave our div
const editor = ace.edit('slackMod-editor');
const session = editor.getSession();
// configure syntax highlighting, theme
session.setMode('ace/mode/css'); // Set lang to CSS
editor.setTheme('ace/theme/dracula'); // Set theme to Dracula
// when we change the content of it, update css
session.on('change', () => {
    updateCustomCSS(session.getValue());
});
```

![Slack Preferences Screen with Custom CSS tab selected with nice syntax highlighted editor](https://i.imgur.com/Ail4shD.png)

## Making CSS Persistent
Looking over GooseMod's Custom CSS code helped a lot with improving our editor, but there didn't seem to be any good hints towards what to do for persistent saving.

### Finding what to use

I asked for help with this in the GooseMod discord and got a single word answer: localstorage.

![SmolAlli — localstorage](https://i.imgur.com/YuSmVcd.png)

Looking into that, I found some great examples in [Mozilla's documentation on localstorage](https://developer.mozilla.org/en-US/docs/Web/API/Window/localStorage). It should be as easy as:

```js
// get
localStorage.getItem("slackMod-CSS")

// set
window.localStorage.setItem("slackMod-CSS", css);
```

We want to have a default value for our custom css; so I did a quick check to see what is returned when a value hasn't been assigned yet.

![>>window.localStorage.getItem("test") -> null](https://i.imgur.com/KZssU7o.png)

### Using it
I started by adding these to our pre-existing get/set methods:
```js
// method to quickly change css
const updateCustomCSS = newCSS => { 
    // update in storage
    window.localStorage.setItem("slackMod-CSS", newCSS);
    // update currently applied CSS
    document.querySelector("#SlackMod-Custom-CSS").innerHTML = "" 
    document.querySelector("#SlackMod-Custom-CSS").appendChild(document.createTextNode(newCSS)); 
}
// method to quickly get inner css
const getCustomCSS = () => { 
    return window.localStorage.getItem("slackMod-CSS")
}
```

I then replaced the code for setting the default contents of our CSS with the following:
```js
if (window.localStorage.getItem("slackMod-CSS") == null) {
    // use default CSS
    styleSheet.innerText = "/*Write Custom CSS here!*/"
    window.localStorage.setItem("slackMod-CSS", "/*Write Custom CSS here!*/")
} else {
    // get saved CSS
    styleSheet.innerText = window.localStorage.getItem("slackMod-CSS")
}
```

That worked!

![Slack Preferences Screen Open](https://i.imgur.com/IV6W6XS.png)

### Small Touchup: Setting Category Icon
I started by measuring all the other Category SVGs and they seemed to all be around 15x15.

Then I went to https://feathericons.com/?query=code, configured it as close to 15x15 as I could (16x), downloaded it, and copied the HTML. I  changed the size to 15x15 by hand 

Finally, I copied the innerHTML of one of the category buttons. With this I deleted the old SVG inside and replaced it with my own, along with changing the span text to Custom CSS:

```js
// Proper Label and Icon
customTab.innerHTML = 
    `<div class="c-tabs__tab_icon--left" data-qa="tabs_item_render_icon">
    <svg xmlns="http://www.w3.org/2000/svg" width="15" height="15" viewBox="0 0 24 24" fill="none" stroke="currentColor" stroke-width="1.5" stroke-linecap="round" stroke-linejoin="round" class="feather feather-code"><polyline points="16 18 22 12 16 6"/><polyline points="8 6 2 12 8 18"/></svg>
    </div>
    <span>Custom CSS</span>`
```

I feel like this tiny change makes a word of difference in how polished the CSS editor feels

![Same Screenshot, Custom CSS now has a icon that looks like < >](https://i.imgur.com/UUxtgQj.png)

## Now What?
I fixed most of the issues outlined in my last blogpost, here is what I have left:
- Selecting custom css in preferences, then selecting a different category leads to an issue
    - I still have absolutely no clue how to fix this. I have spent a solid 3 hours trying different things...
- Injecting is manual and hardcoded to the user's system
    - I think I will make my next blog on this, followed by a tutorial on how to install it on your own slack!