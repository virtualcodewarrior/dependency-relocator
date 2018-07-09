# dependencyRelocator
A small npm post install script that relocates npm production dependencies

## Introduction
### NPM and its problems
These days, more and more JavaScript dependencies rely on NPM as their package manager.
This is in general a good thing because instead of you having to re-invent a package manager which you would have to write, test and maintain,
there already is one out there that gets tested by millions of developers every day and which has a lot of eyes on it.

NPM has a few problems however that are more prominent when hosting a website.

One of them is the "dumping everything in node_modules" problem.
When you are developing you might amass a whole slew of developer only dependencies as well as production dependencies which
all live under the node_modules directory. This directory also contains all your dependencies of your dependencies etc.
You don't want to host all your dev dependencies online and it also might be cumbersome for security audits etc if you cannot easily figure out what is being hosted and what is not.
NPM was never meant to be used for websites in the first place, but since it is being used now, we will have to live with it.

Besides the slew of unneeded hosted (dev)dependencies, there is also the risk of non repeatable releases when relying on a package manager to
provide you with all you production dependencies. It would be safer to have everything required for your production, checked in into github (including the third_party dependencies brought in by npm install).
NPM has introduced package-lock.json to alleviate this problem somewhat, by ensuring that you know which versions of dependencies were exactly used, but you are still relying on NPM to get those versions.

### Solution
For the reasons mentioned above I usually create a ```third_party``` folder under my ```src``` directory, which holds all production third party dependencies, like lodash, webcomponent polyfills etc.
This third_party directory gets added to git to make sure we have total control of the versions of the third party libraries that we use.
It also makes it easier to keep track of what gets hosted, what needs to be audited etc and it prevents us from relative path problems when accessing the third_party libraries.

I still want to use NPM however to add and/or update the third party libraries when needed.
Enter the npmHostedDependencyRelocator post install script.

## npmHostedDependencyRelocator
This script can be run as a post install script for npm install and will copy any dependency listed under the dependencies section of the package.json file, to your own desired location.
It will also recursively go into the package.json files of those dependencies and copy all its dependencies.
The end result is that all production dependencies listed in the package.json file end up in your preferred folder instead which you can then include instead of node_modules.

## usage
Copy the script into the same location as your project's package.json file.
Modify you package.json file to add a postinstall step that calls the script. e.g.
```
{
  "scripts": {
    "postinstall": "node dependencyRelocator.js './src/my dependencies'"
  },
  "devDependencies": {
    "babel-minify": "^0.4.3",
    "babel-preset-minify": "^0.4.3",
    "eslint": "^5.0.1",
    "eslint-plugin-jasmine": "^2.10.1",
    "fs-extra": "^6.0.1",
    "http-server": "^0.11.1",
    "jasmine": "^3.1.0",
    "karma": "^2.0.4",
    "karma-chrome-launcher": "^2.2.0",
    "karma-firefox-launcher": "^1.1.0",
    "karma-jasmine": "^1.1.2",
    "karma-jasmine-es6": "0.0.3",
    "karma-jasmine-html-reporter": "^1.2.0",
    "karma-spec-reporter": "0.0.32",
    "walk": "^2.3.14"
  },
  "dependencies": {
    "@webcomponents/webcomponentsjs": "^2.0.2",
    "jed": "^1.1.1"
  }
}
```

The script has one optional parameter that specifies the target location. If this is not specified it will default to ```./src/third_party```
Once you have modified your package.json file, you re-run npm install and all the dependencies listed in the dependencies section will be copied to the specified target directory.
Note that any old versions or existing directories in the specified target location, will be overwritten.
