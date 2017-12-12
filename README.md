# Electron Tutorial App

### A simple boilerplate applciation to help learn about Electron

<br/>

[![React](/internals/img/react-padded-90.png)](https://facebook.github.io/react/)
[![Webpack](/internals/img/webpack-padded-90.png)](https://webpack.github.io/)
[![Redux](/internals/img/redux-padded-90.png)](http://redux.js.org/)
[![React Router](/internals/img/react-router-padded-90.png)](https://github.com/ReactTraining/react-router)
[![Flow](/internals/img/flow-padded-90.png)](https://flowtype.org/)
[![ESLint](/internals/img/eslint-padded-90.png)](http://eslint.org/)
[![Jest](/internals/img/jest-padded-90.png)](https://facebook.github.io/jest/)
[![Yarn](/internals/img/yarn-padded-90.png)](https://yarnpkg.com/)

Electron is a framework that allows you to build cross platform desktop applications using web technologies using Javascript, HTML, and CSS. It works on handling the hard parts of a desktop application like native menus, debugging, installers, and even updates while allowing you to focus on making your app great. Basically, if you can build a website, you can build a desktop app. Above are some pictures of the different libraries that this simple application uses at its core.

## Setup

* **Note: requires a node version >= 7 and an npm version >= 4.**
* **If you do not already have node installed I suggest using [NVM](https://github.com/creationix/nvm#installation) (node version manager) to install and use different versions of node. A [Windows version](https://github.com/coreybutler/nvm-windows) is also available if necessary**

First, clone the repo via git:

```bash
git clone https://github.com/colingm/slack-social-network.git
```

And then install dependencies with yarn.

```bash
$ cd slack-social-network
$ yarn
```
**Note**: If you can't use [yarn](https://github.com/yarnpkg/yarn) for some reason, try `npm install`.

## Counter Application

Start the app in the `dev` environment. This starts the renderer process in [**hot-module-replacement**](https://webpack.js.org/guides/hmr-react/) mode and starts a server that sends hot updates to the renderer process:

```bash
$ npm run dev
```

You can play around with the app a little bit to see what it does but this tutorial will not be focusing on the javascript side of the app much. The application is a basic counter that allows you to increment and decrement using React. Once you have played around with the app for a bit go ahead and move on through the guide.

#### Developer Tools

When you first start up the app you should see a window appear and at the bottom (or maybe the side) you will see the the Chrome developer tools. Having these tools there allows us to view the console for our application, view network requests, see the sources for our code, and many other things. If the app is in dev mode you can dismiss and reopen these tools at any time using the hotkeys below:

- OS X: <kbd>Cmd</kbd> <kbd>Alt</kbd> <kbd>I</kbd> or <kbd>F12</kbd>
- Linux: <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>I</kbd> or <kbd>F12</kbd>
- Windows: <kbd>Ctrl</kbd> <kbd>Shift</kbd> <kbd>I</kbd> or <kbd>F12</kbd>

## Code

Now that we have got the app running let's take a look at some pieces of the code and the application. A lot of this application is specific to the javascript framework and tools we are using (React and Redux) so we will try to focus on what stands out about electron without worrying too much about what the other tools are doing.

#### Dependencies

Electron uses node's package manager to install the various libraries that an application might need during development and runtime. Like other node projects these dependencies are found in the `package.json` file in the base directory. If you open up this file you will see a lot of different information that might not make sense (as this is a JSON file though the structure is quite simple). At the very top is just some basic information about the application itself but isn't necessary for our app to work. The sections that we care most about are the `scripts`, `dependencies`, and `devDependencies` parts of the file.

```JSON
"scripts": {
  "build": "concurrently \"npm run build-main\" \"npm run build-renderer\"",
  ...
  "dev": "cross-env START_HOT=1 npm run start-renderer-dev",
  ...
  "package-all": "npm run build && build -mwl",
  ...
  "start": "cross-env NODE_ENV=production electron ./app/",
  ...
  "test": "cross-env NODE_ENV=test BABEL_DISABLE_CACHE=1 node --trace-warnings ./test/runTests.js",
  ...
}
```

The `scripts` section of our JSON file allows us to specify any complex action we might need to perform during the development of our application; I have highlighted a few above that are of particular importance. Certain script values can be run directly (like `start` and `test`) but all others we run by typing `$ npm run {scriptName}` as you saw when we started the application.

Near the bottom of the file are the `dependencies` and `devDependencies` these both install packages from the npm repository that will used by our application. The `devDependencies` are libraries that we need during development of the application which might include debug tools, packaging tools, etc. The `dependencies` section has any library that will be needed by our app to run once it has been packaged and shipped off. Luckily NPM takes care of the difference between these two as long as we remember to put things in the right place :)

You might have noticed that there are already a lot of dependencies installed. Our simple application is already using a lot of more advanced tools to make our time developing easier but they are not required for a more simple application. Whenever you need a new library installed you can just come back to this file and add it there.

#### Windows

Just like many other apps, our Electron app has a base location where it can enter and start up. In our case that entry point is in a file called `main.dev.js` that can be found in the `app` folder. Once you open the file scroll until you see `app.on('ready', ...`.

```javascript
app.on('ready', async () => {
  if (process.env.NODE_ENV === 'development' || process.env.DEBUG_PROD === 'true') {
    await installExtensions();
  }

  mainWindow = new BrowserWindow({
    show: false,
    width: 1024,
    height: 728
  });

  mainWindow.loadURL(`file://${__dirname}/app.html`);

  mainWindow.webContents.on('did-finish-load', () => {
    if (!mainWindow) {
      throw new Error('"mainWindow" is not defined');
    }
    mainWindow.show();
    mainWindow.focus();
  });

  mainWindow.on('closed', () => {
    mainWindow = null;
  });

  const menuBuilder = new MenuBuilder(mainWindow);
  menuBuilder.buildMenu();
});
```

This section of code is run inside of electron's main process and it gives some steps to perform once the app reaches a `ready` state. First, the code checks if we are in development mode, if we are it needs to make sure that developer tools are installed and turned on.

The next part is what makes electron cool, we can create a new window for our application. This is not that different from opening up a new window in Chrome (in fact electron uses Chromium) and presents a place for us to run our web code. In this case we are specifying that we want a window that is `1024x728` and that we don't want it to immediately display. We then instruct the window to load our first html file and once we are sure it is loaded we can tell the window to show itself.

Let's try changing the dimensions of our initial window to make it a little larger. Try setting the width to `1600` and the height to `900`, save the file, stop the process for your application, and then rerun `npm run dev`. You should now see your window appear centered on your screen but larger than last time. With electron you can create as many windows as you need and you can size them each differently.

When your window appears it is being shown in a way that is native to your operating system. So for a Mac you will see the three traffic light controls in the top left corner of the window. If you have ever used the Slack app before (Slack is written using electron) you might have noticed that it doesn't have the same title bar. You can actually hide the title bar if you wanted to introduce your own consistent title bar for your application. To make your title bar disappear simply add `frame: false,` right above `width` and below `show` in our window options and then restart the app. Doing this starts our window in a frameless mode which looks pretty slick.


#### Menus

Electron gives you the ability to create menus for your application that can be interacted with just like they can in any other program you might use. The menus for this application are found in `menu.js` which is also in the `app` folder. After opening the file scroll until you find the variable called `subMenuEdit`.

```javascript
const subMenuEdit = {
  label: 'Edit',
  submenu: [
    { label: 'Undo', accelerator: 'Command+Z', selector: 'undo:' },
    { label: 'Redo', accelerator: 'Shift+Command+Z', selector: 'redo:' },
    { type: 'separator' },
    { label: 'Cut', accelerator: 'Command+X', selector: 'cut:' },
    { label: 'Copy', accelerator: 'Command+C', selector: 'copy:' },
    { label: 'Paste', accelerator: 'Command+V', selector: 'paste:' },
    { label: 'Select All', accelerator: 'Command+A', selector: 'selectAll:' }
  ]
};
...
return [
  subMenuAbout,
  subMenuEdit,
  subMenuView,
  subMenuWindow,
  subMenuHelp
];
```

This block of code allows us to define what a menu should look like in the menu bar for our application. In this case we are creating a menu with the `Edit` label that has 6 different actions in it, `undo`, `redo`, `cut`, `copy`, `paste`, and `select all`. If you click the `edit` label on your menu bar you should see the menu appear with those options in it. All of the menus we create are returned from this function after we define all the menus.

Let's create our own menu action and add it to the `view` menu. First, we need to add another import statement to the top of the file. At the top of the file (right after the comment) add the import statement below so the top of your file looks just like this:

```javascript
import Electron from 'electron';
```

This will allow us to use some more parts of the electron api we didn't have access to before. Now find the variable called `subMenuViewDev` because this is where we are going to add our option. Inside the `submenu` array add a comma after the last item in the list (the `Toggle Developer Tools` item) and then the `separator` and `App Menu Demo` items after it so that your menu looks like this:

```javascript
const subMenuViewDev = {
  label: 'View',
  submenu: [
    { label: 'Reload', accelerator: 'Command+R', click: () => { this.mainWindow.webContents.reload(); } },
    { label: 'Toggle Full Screen', accelerator: 'Ctrl+Command+F', click: () => { this.mainWindow.setFullScreen(!this.mainWindow.isFullScreen()); } },
    { label: 'Toggle Developer Tools', accelerator: 'Alt+Command+I', click: () => { this.mainWindow.toggleDevTools(); } },
    { type: 'separator' },
    { label: 'App Menu Demo',
      click: function (item, focusedWindow) {
        if (focusedWindow) {
          const options = {
            type: 'info',
            title: 'Application Menu Demo',
            buttons: ['Ok'],
            message: 'This demo is for the Menu section, showing how to create a clickable menu item in the application menu.'
          }
          Electron.dialog.showMessageBox(focusedWindow, options, function () {})
        }
      }
    }
  ]
};
```

This will first create a separator after the other items and then add a new one that will open a little popup window when we click it. Go ahead and restart your app to see the change. Once it has restarted, click the `View` menu and then click on the new `App Menu Demo` option there to see our new popup window.

#### Keyboard Shortcuts

Now that we have figured out windows and menus, we should look at adding keyboard shortcuts. Global shortcuts are ones that can be detected even when our app doesn't currently have the focus of the keyboard and they allow us to do all sorts of things. We are going to create one of those shortcuts now.

We need to go back to `main.dev.js` to add this shortcut. First, we need to import a couple more things to be able to pull this off. At the top of the file, add the `dialog` and `globalShortcut` imports so that your import statements looks like this:

```javascript
import { app, BrowserWindow, dialog, globalShortcut } from 'electron';
import MenuBuilder from './menu';
```

Now let's add our shortcut. We need to add the shortcut after the app is ready and we also need to make sure to unregister the shortcut when the app is closing so that it isn't called once we are done. Inside the `app.on('ready', ...` block from earlier add this code right after the call to `menuBuilder.buildMenu()`:

```javascript
globalShortcut.register('CommandOrControl+Alt+K', function () {
  dialog.showMessageBox({
    type: 'info',
    message: 'Success!',
    detail: 'You pressed the registered global shortcut keybinding.',
    buttons: ['OK']
  });
});
```

Also don't forget to add the code remove the shortcut, just put this at the end of the file:
```javascript
app.on('will-quit', function () {
  globalShortcut.unregisterAll();
});
```

With these two snippets we have added (and then removed on close) a keyboard shortcut mapped to `cmd+alt+k` that will bring up a little popup window. After you have added this code go ahead and restart the app and try out the new shortcut! This shortcut will work if the app doesn't have focus or even if it has been minimized.

## Finishing Up

We have now played around with some of the cool things you can do with electron but there is really so much more you can do. With electron you can open the system's file manager, open system dialogs, get information about the system, access the clipboard, and lots more. Electron has a lot of power and is already being used by quite a few companies to make consistent native applications.
