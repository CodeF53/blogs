# System Wide Dark Mode
Windows has a lot of residual menus that are light regardless of what you set your Windows theme to.

![Mouse Properties](https://i.imgur.com/OCrVEBb.png)

Some portions of certain windows have been themed, but large amounts of archaic light mode are still present:\
![Control Panel Network Connections menu](https://i.imgur.com/o5kCjhi.png)

Many applications also inherit this inconsistency:\
![Visual Studio Code move file prompt](https://i.imgur.com/uasCvJS.png)

This blog will cover how to fix this issue. The results will look like this:\
![](https://i.imgur.com/5o7WkQ4.png)\
![](https://i.imgur.com/kYB949V.png)\
![](https://i.imgur.com/BaoPMKR.png)

## Custom Theme

### Installing SecureUxTheme
Normally, Windows verifies the signatures of its internal theme files. We need to disable this behavior because we will be using a third party theme.

Start by installing [SecureUxTheme](https://github.com/namazso/SecureUxTheme), a piece of software that removes signature verification of styles in Windows.

I recommend downloading the [the latest release from their GitHub.](https://github.com/namazso/SecureUxTheme/releases)

Run it as an Administrator, and agree to the license.

Make sure "Rename DefaultColors" is checked, then click install on the right hand side:\
![](https://i.imgur.com/ikMAm2d.png)

Then restart your computer:\
![](https://i.imgur.com/FqAPU3m.png)

### Rectify11 Dark Theme
Now that we have a way of installing custom Windows themes, let's get one.

[Rectify11](https://github.com/MishaTy/Rectify11Installer/) is a project that seeks to improve Windows in loads of ways, including custom themes.

Start by downloading and extracting [all their custom themes](https://raw.githubusercontent.com/MishaTy/Rectify11Installer/master/Rectify11Installer/Resources/themes.7z) into a folder:\
![right click > 7-Zip > Extract to "themes\"](https://i.imgur.com/i8cNS5Z.png)

Navigate into the extracted folder, then into the `themes` subfolder, and copy all of its contents to `C:\Windows\Resources\Themes`:\
![](https://i.imgur.com/lYsUiVx.png)

Open the `ThemeTool.exe` again. This time it doesn't have to be as Admin.

Select "Rectify11 dark theme" from the list on the left, check "Ignore cursor", and click apply:\
![](https://i.imgur.com/LCAxdy0.png)

## Mica For Everyone
You will pretty quickly notice that everything except Windows title bars are now dark:\
![](https://i.imgur.com/nS2defu.png)

To fix this, we will use [Mica For Everyone](https://github.com/MicaForEveryone/MicaForEveryone).

Download [their latest release](https://github.com/MicaForEveryone/MicaForEveryone/releases) and run through the setup process.

Once that's done, close the actively running instance from the tray:\
![](https://i.imgur.com/8Tq9Fk0.png)

Then relaunch it as an administrator:\
![](https://i.imgur.com/yvcxSBm.png)

Go back to the tray and open up its settings:\
![](https://i.imgur.com/XR2nN4g.png)

Enable "Run as Administrator on Startup". This is necessary to allow it to style apps that have admin privileges:\
![](https://i.imgur.com/Fgo79ng.png)

Then click on Global Rule, and change Titlebar Color to Dark:
![](https://i.imgur.com/In1SCWx.png)

Now you're done!

## Other Recommendations

- [Improved Cursors](http://www.michieldb.nl/other/cursors/) (Page says Windows 10, can confirm does work on Windows 11.)
