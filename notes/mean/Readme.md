# Creating a blog with the MEAN Stack

* M - MongoDB
* E - ExpressJS
* A - AngularJS
* N - NodeJS

## Assumptions

All terminal commands are assumed to be entered within the `mean-blog` folder on your VM.

## Step 1 - Getting Started

Create a folder called `mean-blog` in the folder that is shared with your Virtual Machine (VM). 

Now, will need to create a `package.json` file to allow use to install node packages. Fortunately NPM has a wizard that will create the `package.json` file for us. In your VM navigate to the folder that you've just created and enter:

```
npm init
``` 

NPM will ask you a series of questions, to accept the default responses just hit enter. Finally, they'll show you a complete version of your `package.json` file, type `yes` to accept it.

As part of the `package.json` file it asked for the app's entry point (default is `index.js`) - let's go ahead and create that file (`index.js`) and fill it with a "hello world":

```js
console.log('Hello World');
```

In your VM, in the `mean-blog` folder, enter `node index.js` into the console - this will run your node app! Congratulations, you've created a NodeJS app. 

## Step 2 - Creating the ExpressJS App

In the terminal:

```
npm install express --save --no-bin-links
```

This install's express and the `--save` flag adds express as a dependency to your `package.json` file while `--no-bin-links` tells npm not to use symbolic links which don't work in shared folders on VM.