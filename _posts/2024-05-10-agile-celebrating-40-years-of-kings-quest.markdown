---
layout: post
title:  "AGILE - Celebrating 40 years of King's Quest"
date:   2024-05-10 06:00:00 +0100
categories: blog sierra agi agile kq1
---

The 10th May 2024 marks the 40th anniversary of King's Quest, the game that started a whole new genre: the animated graphic adventure. To celebrate the event, and as a tribute to the game, I am today releasing [AGILE](https://agi.sierra.games), a way to play Sierra On-Line's AGI adventure games directly in the web browser!

![King's Quest Screenshot](/images/kq1_web_desktop.jpg)

🎂 **Happy 40th Anniversary to King's Quest!!** 🥂

Give it [a try](https://agi.sierra.games)!

Note that this isn't a conversion of the original games into another form, so is quite different to [sarien.net](http://www.sarien.net/) in that regard. AGILE is instead a nearly 100% accurate implementation of the [AGI interpreter](https://en.wikipedia.org/wiki/Adventure_Game_Interpreter) that was used by Sierra to run those games, and being so, it will run the games from the original game files that you may still have lying around somewhere from when you first played them back in the 80s!

Most of the original Sierra On-Line games are still available for purchase online, so for legal reasons, AGILE does not come prepackaged with those games. You must have your own copy and use the import feature to load the game into AGILE. It supports importing from both a folder containing the game, or a ZIP file containing the game. After it has been imported, it will remain in your browser's storage so that you won't have to import it again. To purchase the original games online (should you no longer have them on your bookshelf), check out [gog.com](https://www.gog.com/) and [Steam](https://store.steampowered.com/).

## How to run the games

Start by going to [https://agi.sierra.games](https://agi.sierra.games)

AGILE should run on most modern web browsers. It does, however, rely on some browser APIs that are relatively recent, such as the [SharedArrayBuffer](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/SharedArrayBuffer) and [Origin Private File System](https://developer.mozilla.org/en-US/docs/Web/API/File_System_API/Origin_private_file_system). It has been tested on Chrome, Edge, Firefox and Safari, both the desktop and mobile versions. If it doesn't work for you, then check to make sure that you have updated your browser to the latest version.

It starts by showing the title page, as shown below:

![AGILE Title Page](/images/title_page_web_desktop.jpg)

There is a small question mark icon in the top right that pops up a dialog with a brief description. And take note of the pagination indicator dots at the bottom of the screen. The screen works in a very similar way to the user interface of a mobile device. If you are accessing the website on a touch screen device, then you can swipe to the right to get to the next page. If you are on desktop, you can use the right arrow key, or drag/fling with your mouse, or click on the small right arrow at the bottom of the screen.

The first page to the right has thumbnails for the original Sierra On-Line games. Notice that they are all faded out:

![Sierra Games Page](/images/games_page_web_desktop_faded_out.jpg)

This is to indicate that they are not imported yet. As mentioned above, this is for legal reasons. If you click on one of these, it will open a dialog telling you that you must import your own copy of the game for legal reasons. It also asks you if you would like to import from a ZIP file or a folder. After completing the import process, the thumbnail for the game will no longer be faded out. The game is now imported into the OPFS storage in your browser. If you click on the thumbnail again, it will run the game!

In addition to supporting the import of original Sierra games, AGILE comes pre-packaged with over 100 fan-made AGI games and demos; simply swipe again to the right to see several pages of them:

![Fan-made Games Page](/images/games_page_2_web_desktop.jpg)

These are games that fans of Sierra On-Line built themselves to run on the same AGI interpreter system and so they will therefore also run on AGILE. I have included them so that they and their authors can share in the 40th anniversary celebration. For those games that do not have in-game credits, please refer to the following web page to see who the authors were:

[Fan AGI Release List](http://agiwiki.sierrahelp.com/index.php/Fan_AGI_Release_List_(Sortable))

Be sure to check out [Let Them Eat Cake](https://agi.sierra.games/#/id/ltec), a new fan-made game written by Russ Danner to commemorate the 40th anniversary of King's Quest! It's well worth a play, and is a great tribute to the original game.

## How does it work?

The story of AGILE begins several years ago when I first wrote a version of the [interpreter in C#](https://github.com/lanceewing/agile). That version runs as a desktop application. Development on the new web version of AGILE began in November 2023, in hopes that it would be fully complete by the 10th May 2024, to be a tribute to, and to coincide with, the 40th anniversary of the release of King's Quest.

It started out as a port of the C# AGILE Sierra AGI interpreter project to Java, using the [libGDX](https://libgdx.com/) cross-platform development framework, targeting Desktop, Android and HTML5/GWT. The AGILE interpreter code itself is a straight conversion of the C# code to Java, but for the "AGI Library" part, it is instead using a stripped down version of the [JAGI](https://github.com/lanceewing/jagi) project, which was already written in Java and already had code for loading AGI resources. JAGI was originally written by a mysterious author known as Dr Zoltan and was further extended by myself and Mark Yu.

The reason why the libgdx framework was chosen (and why the JAGI bit is stripped down to the bare minimum), was to get it working as a web app, by primarily targeting the GWT/HTML libgdx platform. [GWT](https://www.gwtproject.org/) (i.e. the Google Web Toolkit) is used by libgdx to transpile the Java code into JavaScript, thus the reason why it is written mostly in Java but is able to run on the web. Some parts had to be written in native JavaScript, where libgdx and GWT didn't provide an out of the box way of doing something.

JavaScript is by default single threaded, which isn't compatible with how AGI blocks waiting for input in some scenarios. The browser tab would simply hang. To address that issue, a web worker was needed, as it allows code to be run outside of the browser's main UI thread. Unfortunately, libgdx and GWT did not provide direct access to that, but by using a project called [gwt-webworker](https://gitlab.com/ManfredTremmel/gwt-webworker) written by Manfred Trammel, I was able to get libgdx to support running the code in a web worker. I then used the SharedArrayBuffer as the primary means of communication between the browser UI thread and the web worker, by both sides having shared simultaneous access to certain parts of the AGI interpreter state.

For those who are interested, the source code can be seen here: 

[https://github.com/lanceewing/agile-gdx](https://github.com/lanceewing/agile-gdx)

I hope you all have fun [trying it out](https://agi.sierra.games) and reminiscing about King's Quest and the other AGI adventure games! 

And finally, a big thank you to our heroes who 40 years ago created King's Quest. To Ken & Roberta Williams, Arthur Abraham, Charles Tingley, Greg Rowland, Ken MacNeill and Doug MacNeill: Thank you for bringing to life such an amazing and enduring concept, and a game and franchise that we will continue to remember for decades to come.
