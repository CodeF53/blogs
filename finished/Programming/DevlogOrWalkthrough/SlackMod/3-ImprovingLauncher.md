## Intro
This is a continuation of my series of blogs documenting my process of making a custom mod for Slack:
- [Initial Creation of the Project](https://dev.to/f53/adding-a-custom-css-menu-to-slack-1090) discusses:
    - Why I am making it
    - How I inject into Slack
    - Problem solving the unique challenges that come with client modding
    - Very basic text editor that is linked to CSS
- [Improving the Editor](https://dev.to/f53/slack-mod-improving-the-editor-4hbn) discusses:
    - Learning from other open source works
    - Adding an editor with Ace
    - `localStorage`, a simple way to make saves persist.

Last blog left off with a finalized custom CSS editor the following note:
> Given my list of issues from last blog, here is what I have left to do:
> - Injecting is manual and hardcoded to the user's system
>   - I think I will make my next blog on this, followed by a tutorial on how to install it on your own slack!

![Slack Preferences menu, with a Custom CSS tab and an nice syntax highlighted editor](https://i.imgur.com/UUxtgQj.png)

## Automatically finding Slack's install location

Because location varies wildly between OS, I started by looking into how to differentiate between user systems, and found `platform.system()` which looks pretty nice.

Looking at [the docs](https://docs.python.org/2/library/platform.html#platform.system), for Windows it returns "Windows", for Linux it returns "Linux", but the docs do not mention what it returns when a user is on a Mac!

Given this, I asked the only Mac user I currently know:

![CodeF53 — Today at 10:03 AM can you install python: https://www.python.org/downloads/ run the following commands in your terminal? ~ python3 > import platform > print(platform.system()) and tell me what prints? There are no docs for what this should do in a mac](https://i.imgur.com/l1StYTU.png)

I got this response:

![screenshot of their terminal running the specified commands, with an output of Darwin](https://media.discordapp.net/attachments/1006243267756691608/1014567865090834472/Screen_Shot_2022-08-31_at_10.10.15_AM.png)

I never would've guessed that, glad I asked.

Given this info, I started off with a simple switch for what OS a user has installed.

```py
# initialize slack location var
slack_location = ""
# find slack location based on system user is running
match system():
    case "Linux":
        # code that gets location for linux
    case "Windows":
        # code that gets location for windows
    case "Darwin": # Mac
        # code that gets location for mac
    case _:
        # happens when user doesn't have windows, mac, or linux
        # tell user their system isn't supported.
```

### Linux

For the Linux case, as much as I would like to write something general, I don't know anyone else that uses Linux that would have Slack installed.

Given this, I just used what worked for my system, along with a little test using `os.path.exists()`.

If it the check fails, I used `input()` to make sure the user sees the error, followed by an exit command.

```py
case "Linux":
    # check if slack is in the location
    slack_location = "/usr/lib/slack/slack"
    if (not exists(slack_location)):
        input("Error Message")
        exit()
```

After a bit of thought, this is the error message I came up with:

```
Could not find Slack install! Please:
    - install Slack
    - open a Github issue so I can support your install location
Press Enter to exit.
```

I wrote this so it could be used as a catchall for whenever any Slack install wasn't found. This was so I could save it to a variable and use it for all not cases of Slack not being found:

```py
# general error to print when slack cant be found
ERR_SLACK_NOT_FOUND =  "Could not find Slack install! Please:\n\t- install Slack\n\t- open a Github issue so I can support your install location\nPress Enter to exit."
# initialize slack location var
slack_location = ""
# find slack location based on system user is running
match system():
    case "Linux":
        # check if slack is in the location
        slack_location = "/usr/lib/slack/slack"
        if (not exists(slack_location)):
            input(ERR_SLACK_NOT_FOUND)
            exit()
...
```

### Windows

Every Slack installation on Windows I have found has been at a directory following this format:
```
C:\Users\[USER_NAME]\AppData\Local\slack\app-[SLACK_VERSION]\slack.exe
```

After some research, I found `os.getenv('LOCALAPPDATA')` which returns:

```
C:\Users\[USER_NAME]\AppData\Local
```
(with [USER_NAME] filled in of course)

Given this, I first made a check for if the user has Slack installed at that location, using the ERR_SLACK_NOT_FOUND message we made earlier:

```py
case "Windows":
    # Check if the %localappdata% location of slack exists
    slack_location = os.getenv('LOCALAPPDATA') + "\\slack"
    if (exists(slack_location)):
        # Code for getting the rest of it
    else:
        input(ERR_SLACK_NOT_FOUND)
        exit()
```

The contents of the Slack directory we are at now look like this:

![contents of C:\Users\F53\AppData\Local\slack, a bunch of sub directories with names: ['app-4.23.0', 'app-4.26.13', 'app-4.26.3', 'app-4.27.154', 'app-4.28.171', 'app-4.9.0', 'packages'] ](https://i.imgur.com/ssIVuVS.png)

For some awful reason, Slack keeps older installations of itself, meaning now we have to do a lot of work to figure out which one of these is the directory we want.

Here's what we have to do in general:
- Get a list of all of the subdirectories in `%localappdata%/slack`
- Filter for only the ones that will have `slack.exe` inside
    - As simple as checking if it contains `"app-"`
- Get the latest version
    - Surprisingly complex problem because version numbers don't work like number numbers
        - `4.26.13` > `4.26.3` > `4.9.0`
    - Luckily, we are making this in Python, so someone will have already written a library for this.

First things first, getting a list of the subdirectories. First google result gives us [a highly voted stackoverflow answer:](https://stackoverflow.com/a/973488)

> However, you could use it just to give you the immediate child directories:
> ```py
> next(os.walk('.'))[1]
> ```

How this code works is pretty complicated to break down.

I'm going to just treat it like a black box, understanding its inputs and outputs.

If this project was one where people really needed to understand every line of the code, I would put it into a well commented function:

```py
# Given a valid directory, outputs an array of sub directory names:
# given an ExampleDirectory with the given structure:
# ExampleDirectory:
# ⎿ subDirectoryA
#   ⎿ subSubDirectory
# ⎿ subDirectoryB
# ⎿file.txt
# Will Return:
# ["subDirectoryA", "subDirectoryB"]
def getSubDirectories(directory):
    return next(walk(directory))[1]
```

But this isn't too big of a project, so I am just going to use it like this:

```py
...
if (exists(slack_location)):
    # Slack keeps old versions for some reason, inject into the latest one.
    # get an array of all the subdirectories
    #   ['app-4.9.0', 'app-4.26.13', 'app-4.26.3', 'app-4.28.171', 'packages']
    subDirs = next(walk(slack_location))[1]
```

To filter these directories, we can use Python's filter method, which removes elements from an array based on what a function returns when given the input.

`filter(function, array)`

Using a lambda for our function looks like this:

`filter(lambda dir: "app" in dir, subDirs)`
- `"app-" in "app-4.23.0"` => True
    - Keeps in array
- `"app-" in "packages"` => False
    - Removes from array

Technically, `filter()` returns a "filter object" so we have to wrap it in `list()` to cast it back to something useful.

Implementing this gives us the following function:

`list(filter(lambda dir: "app" in dir, subDirs))`

```py
# Slack keeps old versions for some reason, inject into the latest one.
# get an array of all the subdirectories that have app-
#   ['app-4.9.0', 'app-4.26.13', 'app-4.26.3', 'app-4.28.171']
subDirs = list(filter(lambda dir: "app-" in dir, next(walk(slack_location))[1]))
```

With this new array, all we need to do is get the latest version. Which should be pretty easy because Python has libraries for everything.

First Google results said `packaging.version.Version()`.

In order to test if this works, I imported the library and initialized a test array inside the Python console:

```py
>>> from packaging.version import Version
>>> subDirs = ['app-4.9.0', 'app-4.26.13', 'app-4.26.3', 'app-4.28.171']
```

To run it on every element on the array, we can use `map(function, array)`:
```py
>>> map(Version, subDirs)
<map object at 0x7f0d44e7f2b0>
```

Looks like similarly to `filter()`, we need to cast `map()`'s output to a list so its useful:

```py
>>> list(map(Version, subDirs))
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
  File "/usr/lib/python3.10/site-packages/packaging/version.py", line 266, in __init__
    raise InvalidVersion(f"Invalid version: '{version}'")
packaging.version.InvalidVersion: Invalid version: 'app-4.9.0'
```

Looks like we need to do a bit of pre-processing, as this library cant handle strings that aren't just version numbers.

To clean up our strings we can do `string.replace("app-","")`:

```py
>>> "app-4.9.0".replace("app-","")
'4.9.0'
```

To do that over our whole array, we can use map:
```py
>>> list(map(lambda dir: dir.replace("app-","") ,subDirs))
['4.9.0', '4.26.13', '4.26.3', '4.28.171']
```

Trying `Version` again with our cleaned strings:
```py
>>> list(map(lambda dir: Version(dir.replace("app-","")),subDirs))
[<Version('4.9.0')>, <Version('4.26.13')>, <Version('4.26.3')>, <Version('4.28.171')>]
```

Now we finally have an array of this library's version object.

Lets call `max()` on it to get the latest version:

```py
>>> max(list(map(lambda dir: Version(dir.replace("app-","")),subDirs)))
<Version('4.28.171')>
```

Looks promising, lets change our input data and see if it still works:

```py
>>> subDirs = ['app-4.28.171', 'app-4.26.3', 'app-4.26.13', 'app-4.9.0']
>>> max(list(map(lambda dir: Version(dir.replace("app-","")),subDirs)))
<Version('4.9.0')>
```

Looks like this library cant even do what it promised.

That may have felt like a massive waste of time, but it would've been a nightmare to debug that problem when it happened in the actual code.

After Googling again, I found a different library: `pkg_resources.parse_version()`. Lets test it! Luckily, we can reuse a lot of the code we just used to test the previous library.

The first disappointing part of the previous library was it not being able to parse our strings while they still had `app-` in them. So I decided to first test if our new library needs the same cleanup:

```py
>>> from pkg_resources import parse_version
>>> list(map(parse_version, subDirs))
[<LegacyVersion('app-4.28.171')>, <LegacyVersion('app-4.26.3')>, <LegacyVersion('app-4.26.13')>, <LegacyVersion('app-4.9.0')>]
```

Great, less parsing overall needed for us!

But does it actually compare these correctly? Lets test:

```py
>>> max(list(map(parse_version, subDirs)))
<LegacyVersion('app-4.28.171')>
```

YES!

Alright, let's implement this into our code!

Putting in what we have looks like this:

```py
...
# ['app-4.9.0', 'app-4.26.13', 'app-4.26.3', 'app-4.28.171']
subDirs = list(filter(lambda dir: "app-" in dir, next(walk(slack_location))[1]))
# a "LegacyVersion" object of the newest version in subDirs
# <LegacyVersion('app-4.28.171')>
latest_version = max(list(map(parse_version, subDirs)))
```

Now what? For me, finding how to get the latest version took so long that I forgot what we even needed it for.

The whole point of this was to get which Slack subdirectory to use, after remembering this, I realized a `LegacyVersion` object isn't very useful as there is no easy way to turn it back into the correct subdirectory.

Given this, I put our array of `LegacyVersion` objects into a variable instead:

```py
# array of comparable "LegacyVersion" objects in the same order as subDirs
versions = list(map(parse_version, subDirs))
```

Then, I got the index of the latest version in that array:
```py
versions.index(max(versions))
```

Because `versions` has the same order as `subDirs`, we can use that index in our `subDirs` array to get the directory containing the newest version of Slack:

```py
# the directory containing the newest version of slack:
subDirs[versions.index(max(versions))]
```

With that, we can finally get the full directory to the latest Windows version of Slack.

```py
# Check if the %appdata% location of slack exists
slack_location = os.getenv('LOCALAPPDATA') + "\\slack"
if (exists(slack_location)):
    # Slack keeps old versions for some reason, we want to inject into the latest one
    # get an array of all the subdirectories that have app-
    # ['app-4.9.0', 'app-4.26.13', 'app-4.26.3', 'app-4.28.171']
    subDirs = list(filter(lambda dir: "app-" in dir, next(walk(slack_location))[1]))
    # array of comparable "LegacyVersion" objects in the same order as subDirs
    versions = list(map(parse_version, subDirs))
    # The full path to the latest version of Slack's slack.exe
    slack_location = '"' + slack_location + "\\" + subDirs[versions.index(max(versions))] + "\\slack.exe" + '"'
else:
    # If Slack is not found in windows appdata
    print(ERR_SLACK_NOT_FOUND)
    exit()
```

### MacOS
I don't have a Mac, so this is going to be a pain.

I started by asking the one person that uses a Mac and Slack if he knew where Slack is installed:
![CodeF53-asks with examples Big Sister - Computer/Applications Slack.app](https://i.imgur.com/6vKP5KQ.png)

Given this, I asked him to try running a few commands in his Python terminal as a test to see if what I was about to do would work.

![CodeF53-asks to run exists("/Computer/Applications/Slack.app"), Big Sister replies with a sad emoji](https://i.imgur.com/WmtfPBo.png)

After that, I was tempted to just give up on MacOS and use the following code:

```py
case "Darwin": # Mac
    input("MacOS is not supported yet\n\tpress any key to exit")
    exit()
```

But I wasn't satisfied with that.

I thought about it some more, realized MacOS may just not display the full path to the user, and asked my friend to `cd` to `/Computer/Applications/` and run `pwd` (print working directory).

![what I just said, them saying /Applications, then me following up with a new test code](https://i.imgur.com/OULfoNV.png)

His response took a bit, but it was worth it:

![A screenshot of the word "TRUE" heavily deepfried](https://i.imgur.com/YlVrs4V.png)

With that, I knew at least this bit of code would work:

```py
case "Darwin": # Mac
    # check if slack is in the location
    slack_location = "/Applications/Slack.app"
    if (not exists(slack_location)):
        input(ERR_SLACK_NOT_FOUND)
        exit()
```

I pushed my changes and asked my friend to try using my mod.

He sent back a stacktrace, notably containing this:
```
Traceback (most recent call last):
  File "/Users/thelightdisk/Development/SlackMod/slack_launch.py", line 72, in <module>
    download(libURL)
```

...meaning there was some error with my download code.

To narrow down the issue, I asked him to try and run some more stuff in the Python terminal as a test:
```py
~ python3
> from wget import download
> wget("https://ajaxorg.github.io/ace-builds/src-min-noconflict/ace.js")
```

In response, he sent back an error containing the exact same issue:

![a stacktrace](https://i.imgur.com/JmXScC4.png)

Given this, I knew there was some issue with `wget.download()` on MacOS.

I am going to skip over around 2 hours of me having him try different download libraries in an attempt to find one that worked on mac.

Eventually I came to the conclusion that I should just do this:

```py
# macOS is stupid and doesn't like wget's download
def download(url):
    if (system()=="Darwin"):
        fileName = url.split("/")[-1]
        sysrun(f"curl -o { fileName } \"{url}\"")
        sleep(0.1)
    else:
        wgetDownload(url)
```

After that, all of the downloading part of the script worked, but then it spat out an error that `Slack.app` is a directory.

![/Applications/Slack.app is a directory](https://i.imgur.com/5nsHVzp.png)

I asked my friend to get the folder structure with `ls` and he did one better:

![The folder structure inside Slack.app](https://media.discordapp.net/attachments/1006243267756691608/1014661985780121651/Screen_Shot_2022-08-31_at_4.24.11_PM.png?width=440&height=314)

Given this, I updated the Mac `slack_location` one last time:

```py
case "Darwin": # Mac
    # check if slack is in the location
    slack_location = "/Applications/Slack.app/Contents/MacOS/Slack"
    if (not exists(slack_location)):
        input(ERR_SLACK_NOT_FOUND)
        exit()
```

With that change, it finally worked:

![screenshot of discord of a person's screenshot of the custom css menu](https://i.imgur.com/pXcFyQD.png)

## Automatically Killing Slack

Throughout having friends test the mod in the prior step, them forgetting close Slack when running the script was a pretty common issue.

Given that, I figured it would be a good idea to have the script do it automatically.

I found some solid looking code for [killing a process with Python on StackOverflow](https://stackoverflow.com/a/67509457/8133370), so I modified it a slight bit and gave credit in a comment:

```py
# Kill Slack
# Mac and Linux have no extension on executables
process_name = "slack"
if (system()=="Windows"):
    process_name = "slack.exe"
# https://stackoverflow.com/a/67509457/8133370 under (CC BY-SA 4.0)
try:
    print(f'Killing Slack Processes')
    for process in process_iter():
        try:
            if process_name == process.name() or process_name in process.cmdline():
                args = process.cmdline()
                if (len(args) > 4):
                    args = args[0:3]
                print(f'\tkilling instance - {args}')
                process.terminate()
        except Exception:
            print(f"\t\tPermission Denied, Moving on.")
except Exception:
    print(f"Failed to get processes to kill, assuming Slack isn't open")
```

Note that instead of printing an exception stacktrace I went for 2 generic errors. This is because the following errors occur on Windows when there are no real problems:

On occasion, when running the launch script on Windows without Slack open, it triggered the outer try statement. Given this, I replaced the error with this:
```
Failed to get processes to kill, assuming Slack isn't open
```

On Windows, there are some Slack processes that have elevated privileges and can't be killed by our script. When we try to kill them, it triggers the inner try statement. Thing is, these processes are automatically closed when we kill the non-elevated processes.

Given that it really doesn't matter we can't kill them, I replaced the error with this:
```
Permission Denied, Moving on.
```

Other than those small tweaks, the script from StackOverflow worked perfectly on all systems I tested.

## Conclusion
### Remaining issues
After all of that, using SlackMod is still a bit of a pain.

To install it, you have to:
1. Download Python 3
2. Download the mod from GitHub
3. Install the needed Python libraries

After you do that, every time you want to open it, you have to do the following:
1. Open a terminal
2. Navigate to wherever you installed the mod
3. Run `python3 slack_launch.py`

Eventually, I will attempt to address this, but solving this problem has proved VERY difficult.

### Meta and Plan for next Blogs
I originally wanted this to be a short blog covering the process of making installing and using the mod easier. It turned out that just fixing the launch script was a royal pain, but that has been complete for nearly a week.

I have spent the last week trying to find a neat way to make the mod easier to launch, while also making it easier to install. I have completely failed at that.

If you care, here is a quick summary of all the ideas I have gone through:
- Make adding an OS-specific script to OS's app launch directory part of the "install process".
    - Adds a lot of effort to setup process
    - Hard to explain
    - Varies wildly by OS
        - Hundreds of ways to do this on Windows, all of which feel wrong.
- Making a separate OS-specific script that automatically does the above.
    - Turns out to be so complex I don't know how to get into it.
- Compiling python script into 3 OS-specific executable files
    - Still painful to add to OS app launch directory
    - Compiling Python is PAIN
    - How the hell will `electron-inject` see the JS?

Next blog will likely be some final cleanup to our injected JavaScript, which should be quickly followed by an "official" v1 release blog with a detailed install tutorial.