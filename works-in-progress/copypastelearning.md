## Intro
If you are taking any programming course, you probably frequently copy paste commands from course assignments into your terminal. This is just a couple of tips so following along with these is less of a pain.

## Quick copy pasting
When possible, viewing your lesson's `readme.md`s on github is preferred, as they add a nice button for copying the contents of a code block, which cuts out the pain of manually selecting all of a code block's contents.

![a mouse cursor over a copy button in a github codeblock](https://i.imgur.com/7d1d4XT.png)

## Command prefix fix
### The problem:
Code blocks that you run in a terminal are prefixed with a $ to indicate where you run it.
```sh
$ echo "example command"
```

This has the side effect of making them not work nice with copy pasting, as `$` isn't a command:

![zsh terminal screenshot, command run `$ echo "example command"`, outputs `zsh: command not found: $`](https://i.imgur.com/TjkRrr5.png)

### A solution:

To fix this, you can run

```sh
$ () {"$@"}
```
(<small>assuming you are in a good shell</small>)

![zsh terminal screenshot, command run `$ (){"$@"}`, command run `$ echo "example command"`, outputs `example command`](https://i.imgur.com/WXkIncO.png)

### How does this work?

basically, we are creating a function with the name `$`:
```
functionName (args) {shell command to run}
```

`$@` is a reference to all of the arguments passed into the function. We have to pass it into quotes because it may have spaces inside.

This makes it so when you run a command in your shell starting with `$`, you are actually running a function that runs whatever arguments you pass into it as a shell command.

### Making this persistent

But this isn't persistent, you have to run it again every time you restart your terminal!

To make it persistent, run this following command:
```sh
cd ~; echo '\n# Makes copy pasted commands with a $ in front of them still work\n$ () {"$@"}' > .bashrc
```

This goes to your home directory, and adds the following 2 lines of code to the end of `.bashrc`:
```sh
# Makes copy pasted commands with a $ in front of them still work
$ () {"$@"}
```
All code added to `.bashrc` is run whenever you start your terminal. So running this command makes it so `$` always acts like a function that runs it's arguments