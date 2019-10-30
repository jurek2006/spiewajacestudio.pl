# spiewajacestudio.pl

## Scripts for development, build, deploy

There are few scripts in package.json prepared for development (live watching changes), build (generating page for gh-pages, used for new version's testing) and deploy (creates version for production deployment on the server).

### npm start (live development)

`npm start`

Starts live server with watching and bundling scss to css, and html (there are html partials used)

### npm run build (build for gh-pages)

To build gh-pages (previev & testing version) it is only needed to build the current version:

`npm run build`

& push the changes in master branch

The build command will build current version to docs/ folder (if not changed in package.json > config > build folder) which is used by gh-pages

### npm run deploy (build for deployment on production server)

To build on production use:

`npm run deploy`

it creates development version (by default in folder deploy/DEPLOY_OUTPUT, it can be changed in package.json > config > deployOutputFolder)

It creates deployment version - content from DEPLOY_OUTPUT folder should be send to the production server.

Different from development & build versions - during the process - **Google Analytics Tag is included** in head section of the page (See more in part Google Analytics Tag below).

When the page is built with deploy script, at the end of it, you can find a comment like this:

`<!-- DEPLOY 2019-10-30_11:54.742 -->`

It shows when the version was build for production.

## Scripts details (package.json)

#### Folders configuration

You can find defined folders in object:

```
"config": {
        "liveFolder": "live", //folder for development live server
        "buildFolder": "docs", //folder for preview & testing (on gh-pages, remember push on master branch)
        "deployWorkingFolder": "deploy", //folder for deployment process - can't be erased
        "deployOutputFolder": "deploy/DEPLOY_OUTPUT" //folder where you find the result for production deployment
    },
```

#### npm start

It bundles html and css from scss, runs watching changes for both and starts live-server.

`npm start`

**package.json - spiewajacestudio.pl**

-   "start": **prepares everything for live-server to watch changes during development - stores generated filed in /life folder - if not changed, source is in src/**
    -   "live:pre": **does the following:**
        -   "live:pre:version": **creates file with timestamp which will be added at the end of index.html**
        -   "live:pre:mkdirs": **creates folders for css, assets and favicon**
        -   "live:pre:copy-assets": **copies the whole content of img/**
        -   "live:pre:copy-favicon": **copies the whole content of favicon/**
    -   "live:sass": **compiles scss styles to style.css**
    -   "live:sass:watch": **compiles scss styles to style.css - watching live**
    -   "live:html:watch": **compiles html with partials to html - watching live**

#### npm build

-   "build": **creates build version (preview and testing) for gh-pages, doing following:**
    -   "build:pre": **does the following:**
        -   "build:pre:version": **creates file with timestamp which will be added at the end of index.html**
        -   "build:pre:clean": **erases whole folder for build output**
        -   "build:pre:mkdirs": **creates folders for css, assets and favicon in build output folder**
        -   "build:pre:copy-assets": **copies the whole content of src/img/ to img/ in build output folder**
        -   "build:pre:copy-favicon": **copies the whole content of src/favicon/ to favicon/ in build output folder**
    -   "build:css": **does the following:**
        -   "build:css:compile-sass": **compiles scss styles to css in src/css**
        -   "build:css:concat-css": **concatenates css files (at the moment not in use)**
        -   "build:css:autoprefix": **adds autoprefixes**
        -   "build:css:compress-css": **compresses css and stores the result in the build output folder**
    -   "build:html": **includes partial html files from src and stores the result in the build output folder**

#### npm deploy

    Builds deploy version - ready to be uploaded on production server.
    It adds special html partials prepared only for deployment version (from deploy/deploy_helpers/only_deploy_html_partials/ if not changed in folders' configuration ).
    Currently used for adding Google Analytics Tag only in the production version.
    Copies also all files from deploy/deploy_helpers/only_deploy_other to the root of the deployment output folder (used for .htaccess)

-   "deploy": **builds deploy version**
    -   "deploy:pre": **does the following:**
        -   "deploy:pre:cleanOutpur": **erases while folder for deploy build result**
    -   "deploy:html": **does the following:**
        -   "deploy:html:mkdirs-temp": **creates a temporary folder deployTemp to compile html with partials in it**
        -   "deploy:html:copy-html": **copies all html files (html and html partials \*) from src folder (only, child folders not included) to the temporary folder**
        -   "deploy:html:version-update": **creates file with timestamp which will be added at the end of index.html - indicating that this is deployment version**
        -   "deploy:html:copy-only-deploy-html": **copies partials (and possible any other files) to be included only in development version (different versions were in build and development)**
        -   "deploy:html:compile": **includes partial html files from the temporary folder (including special 'only-deployment' partials) and stores the result in the deployment output folder - it is the final version of html file**
        -   "deploy:html:clean-temp": **deletes the temporary file as it is not needed anymore**
    -   "deploy:css": **does the following:**
        -   "deploy:css:mkdirs": **creates a css folder in the in the deployment output folder**
        -   "deploy:css:compile-sass": **compiles scss styles to css in src/css**
        -   "deploy:css:concat-css": **concatenates css files (at the moment not in use)**
        -   "deploy:css:autoprefix": **adds autoprefixes**
        -   "deploy:css:compress-css": **compresses css and finally stores the result in the deployment output folder**
    -   "deploy:assets": **makes copy of assets (img/) from src to the deployment output folder, following:**
        -   "deploy:assets:mkdirs": **creates folder for assets**
        -   "deploy:assets:copy": **copies the whole content of src/img/ to img/**
    -   "deploy:favicon": **makes copy of favicon folder from src to the deployment output folder, following:**
        -   "deploy:favicon:mkdirs": **creates folder**
        -   "deploy:favicon:copy": **copies favicon content**
    -   "deploy:copy:only_deploy_other": **copies all files from deploy/deploy_helpers/only_deploy_other to the root of the deployment output folder (used for .htaccess)**

## Use html partials

You can use html partials (similarly as scss partials) ([html-include](https://www.npmjs.com/package/html-includes) is used).

You can include it like this:

**index.html - spiewajacestudio.pl**

```...
${require('./_somePartial.html')}
```

### Html partials only for deployment version

If you want to include some partials **only in production deployment version** you can put the partial in **deploy/deploy_helpers/only_deploy_html_partials/** and then include it in html:

**index.html - spiewajacestudio.pl**

```...
${require('./only_deploy_html_partials/_googleAnalytics.html')}
```

To work properly - there must be "placeholder" in a file with the same name in **src/only_deploy_html_partials/** (otherwise there will be a script error during development/build)

## Google Analytics Tag

To provide Google Analytics Tag - just store it in (as you copy it from GA) **deploy/deploy_helpers/only_deploy_html_partials/\_googleAnalytics.html**

Automatically it will be added only in the production deployment version (built with `npm run deployment`).

(You should also define "placeholder" for development and build versions in **src/only_deploy_html_partials/\_googleAnalytics.html**)

## .htaccess and other files for development

To provide any files we want to include in deployment's version root folder - we store these files in **deploy/deploy_helpers/only_deploy_other**. The files will be copied automatically.

Now it is used for .htaccess
