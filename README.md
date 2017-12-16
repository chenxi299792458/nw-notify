# nw-notify
*Nice and simple notifications for node-webkit apps*

## DISCONTINUED - Contact me if you want to take over this repo

**Version 1.0.0 has breaking changes, see section "Changelog" below.**

![Mac demo](https://github.com/cgrossde/nw-notify/raw/gh-pages/nw-notify-mac-small.png)
![Win demo](https://raw.githubusercontent.com/cgrossde/nw-notify/gh-pages/nw-notify-windows-small.png)

*nw-notify* displays notifications in the lower right corner. Notifications are stacked (most recent on the top) and slide down, once they expire. *nw-notify* is a child of [Pullover](https://github.com/cgrossde/Pullover), a destop client for the Pushover service. 

## Features

* Windows and Mac supported (Linux not tested, but should work)
* AppIcons (optional, left of notification text) and images (optional, right of notification text)
* Sounds
* Close button (top right corner)
* Open URLs (optional)
* Callbacks for `show`, `click` and `close`
* Queues notifications if not all can be shown at once

## Usage

```JavaScript
var nwNotify = require('nw-notify');
// Change config options
nwNotify.setConfig({
    appIcon: nwNotify.getAppPath() + 'images/icon.png',
    displayTime: 6000
});

// Send simple notification
nwNotify.notify({ title: 'Notification title', text: 'Some text' });
// Notification with URL, click notification to open
nwNotify.notify({ title: 'Notification title', text: 'Some text', url: 'http://wikipedia.org'});
// Or with image and playing a sound on show
nwNotify.notify({ 
    title: 'Notification title', 
    text: 'Some text', url: 'http://wikipedia.org',
    image: nwNotify.getAppPath() + 'pathTo/image/from/nwAppRoot/folder.png',
    sound: nwNotify.getAppPath() + 'sound.wav'
});
// Do something when user clicks on notification
nwNotify.notify({ title: 'Custom func', onClickFunc: function() {
    // Your code here
    console.log('User clicked notification')
}});

// Change config options between notify calls
nwNotify.setConfig({
    appIcon: nwNotify.getAppPath() + 'images/otherIcon.png',
    defaultStyleText: {
        color: '#FF0000',
        fontWeight: 'bold'
    }
});
// Send notification that uses the new options
nwNotify.notify({ title: 'Notification title', text: 'This text is now bold and has the color red' });

// See below for more options
```

## Function reference

### notify(notificationObj)
Display new notification. For possible properties see example below:

~~~
notify({
    title: 'Title',
    text: 'Some text',
    image: 'path/to/image.png',
    url: 'http://google.de',
    sound: nwNotify.getAppPath() + 'notification.wav',
    onClickFunc: function() { alert('onCLick') },
    onShowFunc: function() { alert('onShow') },
    onCloseFunc: function() { alert('onClose')}
});
~~~

For more info on the `onClickFunc`, `onShowFunc` and `onCloseFunc` callbacks see below.
There are two **sound files** provided in the `sounds/` folder. They are free to use (even commercially) under the specified licenses at the end of this document.

### setConfig(configObj)
Change some config options. Can be run multiple times, also between `notify()`-calls to change *nw-notify*s behaviour.

### getAppPath() : string
Returns path to root of your node webkit app. Use it to provide paths to app icon or image files that are shipped with your app.

### closeAll()
Clears the animation queue and closes all windows opened by *nw-notify*. Call this to clean up before quiting your app. Not needed with `config.autoCleanup` enabled (default).

### setTemplatePath(path)
If you want to use your own `notification.html` you use this method. Use it like this: `nwNotify.setTemplatePath(nwNotify.getAppPath() + 'path/to/notification.html');`

### calcMaxVisibleNotification() : int
Returns the maximum amount of notifications that fit onto the users screen.

## Max notifications and queueing

On startup *nw-notify* will determine the maximum amount of notifications that fit on the screen. This value will be stored in `config.maxVisibleNotifications` but cannot be greater than 7. This is to ensure that all animations go smoothly and *nw-notify* does not freeze your computer. However you can overwrite this value with `setConfig()`. If you do that you should use `calcMaxVisibleNotification()` to check if that many notifications fit onto the users screen.
**Queueing:** Once the limit of `config.maxVisibleNotifications` is reached, *nw-notify* will queue all new notifications internally. The order of `notifiy()`-calls will be preserved and once old notifications fade out, the queued notifications are shown.

## Callbacks

Calling `notify()` will return an unique id for this particular notification. Each callback (`onClickFunc`, `onShowFunc`, `onCloseFunc`) will return an event object which contains the notification id, the event name(click, show, close, timeout, closeByAPI) and a function to close the notification:

```JavaScript
{
    id: 32,
    name: 'click',
    closeNotification: function() {}
}
```

**Example**
```JavaScript
function handleClick(event) {
    console.log('User clicked notification ' + event.id + '. Closing it immediately.');
    event.closeNotification();
}

function handleClose(event) {
    console.log('Notification was closed because: ' + event.name);
}

nwNotify.notify({
    title: 'Notification title',
    text: 'Some text',
    onClickFunc: handleClick,
    onCloseFunc: handleClose
});
```

## Config options
See [the wiki](https://github.com/cgrossde/nw-notify/wiki/Config-defaults).

## Changelog

### change
* code compatible nwjs-sdk-v0.27.1-win-x64, and bug fixed when call `closeAll()`, remove Autocleanup feature.

### 1.0.0

**Breaking changes:**

 * The use of `notify(title, text, ...)` has been removed. Please call `notify()` with an object: `notify({ title: 'some title', text: 'some text' })`.
 * `config.animateInParallel` is now set to true (because of better performance)
 * Autocleanup feature was added and enabled by default. Calling `closeAll()` is not needed anymore. It will be called automatically when the main window is closed.

**New features:**

 * Notification sounds: `nwNotify.notify({title: 'notification with sound', sound: nwNotify.getAppPath() + 'notification.wav'})`. (Thanks [@makkesk8](https://github.com/makkesk8))
 * Autocleanup windows (no need to call `closeAll()` anymore) `config.autoCleanup`. (Thanks [@makkesk8](https://github.com/makkesk8))

**Changes:**

* Fixed bug where `getAppPath()` would not work when the window's URL contained anchors `#`
* More robust way of accessing the default `notification.html` template. If it can not be found we create one in the working dir and use that. Useful with webpack because in those setups the `node_modules` folder is not present.
* Catch errors of `setStyleOnDomElement()` and report them to the user

### Older versions

**0.2.3 - Fixes [#16](https://github.com/cgrossde/nw-notify/issues/16)**

**0.2.2 - Primary screen detection**

**0.2.0**

 * Return events with callbacks (events contain name, id of notification and the `closeNotification` function to close the notification early or programatically)
 * Now under MIT License
 * Callbacks for *show*, *click* and *close*
 * Pass options as object to `notify()`

**0.1.1 - Readme update**

**0.1.0 - Initial version**


## License

    The MIT License (MIT)
    
    Copyright (c) 2014 Christoph Groß <gross@blubyte.de> (http://chris-labs.de)
    
    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:
    
    The above copyright notice and this permission notice shall be included in
    all copies or substantial portions of the Software.
    
    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN
    THE SOFTWARE.

* **sounds/sound_1:** [Taken from RCPTONES.com](http://rcptones.com/dev_tones/) Licensed under Creative Commons Attributen License (http://creativecommons.org/licenses/by/3.0/us/)
* **sounds/sound_2:** [Taken from Freesound.org](https://www.freesound.org/people/GameAudio/sounds/220212/) Licensed under the Creative Commons 0 License (http://creativecommons.org/publicdomain/zero/1.0/)
