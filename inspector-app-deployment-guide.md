---
description: Instructions for installing, deploying and maintaining the Inspector App
---

# Inspector App Deployment Guide

## Preliminaries

1. Inspector App is an Shiny app for visual inspection of measurement data. The app  can be configured for different types of measurements. The app can be deployed in several ways. Deployment and configuration demands good knowledge of R/Shiny (notably Shiny Modules and R6 classes) and the of the platform you want to deploy on.&#x20;
2. Access to the GitHub repo [UtrechtUniversity/isoinspector](https://github.com/UtrechtUniversity/isoinspector). Because you are probably making your own configurations and additions to the software, it is adviced to first fork the repo and then to git clone the repo onto your local workstation. E.g as a project in RStudio.
3. But, if you are satisfied with the app as is and want to run the app locally, you only have to install the package directly from GitHub: `devtools::install_github(repo = 'UtrechtUniversity/isoinspector)`.
4. The app-as-package concep is from the book  [Engineering Production-Grade Shiny AppsInspector App Architecture](https://engineering-shiny.org/index.html) and associated R package [{golem}](https://github.com/ThinkR-open/golem). However, the methodology is not obediently followed.
5. If you want to deploy your app on shinyapps.io, you need a token/secret pair. If your organisation has a Shinyapps.io  account, you must  apply for a token by your administrator. You can also get a private account at small fee or no fee at all. To manage your token and to upload an app you must install package {rsconnect} on your workstation. See for more info about [shinyapps.io](https://docs.rstudio.com/shinyapps.io/index.html).
6. The app can also be ran in a Docker container. This guide tells you how to produce a appropriate Docker file, but assumes you have right knowledge and skills to manage a Docker Engine, etc.

## Architecture of the inspector app

![Main building blocks of inspector app](<.gitbook/assets/architecture (1).jpg>)

Contrary to most Shiny apps,  this app makes use of explicitly triggered events  implemented by [{gargoyle}](https://github.com/ColinFay/gargoyle) and a central [R6 class](https://adv-r.hadley.nz/r6.html) object to implement a more event-driven program instead of relying on _reactivity_ alone. The central Inspector Object holds the data, the changes to the data, and the applied filters and brushes (= selections in interactive plots). The R6 class is documented in the package.

```
library(isoinspector)
?InspectorObj
```

The app has four main modules that provide the user the tools to read/save the data, to filter the data, to visually inspect the data and to edit the data. All the modules exchange data with the central inspector object. The logical modules are implemented by real [Shiny modules](https://shiny.rstudio.com/articles/modules.html). These modules are documented in the package.

```
library(isoinspector)
?inspector_read
?inspector_edit
?inspector_inspect
?inspector_edit
```

###

### Configuration of the Inspector App

Which variables to use for filtering, for plotting, for editting etc. are specified by a configuration. A configuration can be internal package data, but also json files in the working dir of an app. Which configuration to use is specified at start-up time: run\_inspector(config = 'myconfig'). 'myconfig' is the name of the internal variable or the name of the json file. See [here](inspector-app-deployment-guide.md#how-to-configure-an-inspector-app) for informations about adding/changing configurations.

In the package the elements of a configuration are documented:

```
library(isoinspector)
?configuration
```

### &#x20;Inspector App and its environment

![](.gitbook/assets/app\_env.jpg)

Typically, an inspector app shall be used as part of a data processing pipeline/workflow.  Pre-processed and labeled data from devices will be presented in the app for visual inspection. The user can add annotations, comments, etc. These changes must often be fed back into the workflow. This loop is  is not fully developed yet, as the precise integration with the workflow is not known at the moment.&#x20;

As you probably already know, a Shiny app has two parts:

* A user interface (UI) that runs in the browser on the local workstation of the user
* A server that runs inside . an R process on some platform. That platform could be shinyapps.io, R Shiny Server, or a Docker Swarm. But an R proces on the local workstation can also be used as a "server platform".

Currently the inspector app always uploads input data via the browser's file upload. So users can only use data that is on a storage device which is connected  to their local workstations.&#x20;

#### Ouput files

When the user ends a session she gets the option to download the output data to her local workstation. The output data is the input data (all the records) but with the modifications applied. The user can also opts for only downloading the changes.

The apps also stores in its working directory an output file  `output-username-timestamp.rds`. That file only contains the modified records of the input data. Be aware that users often don't have  access to the working directory of the app when the app runs on a server.

#### Recovery files

During a session the app maintains a backup. If the user inadvertently ends a session (e.g. closing browser window), these files will be used to recover the previous situation. Therefor the new session has to start in the same working directory. **In cloud services like `shinyapps.io` that is not always the case.**&#x20;

#### Configuration files

When the user is in an session she can download the current configuration. This json file can be used as a template for making adjustments to the app (e.g. adding plots or filters). If the user runs the app on the local workstation the app can reads the new configuration file when placed in the working directory. Otherwise the system administrator has to be called in. See [here](inspector-app-deployment-guide.md#how-to-configure-an-inspector-app) for more info.

**The configuration download option is currently disabled, because the configurations contains user data. When authentication is handled the way it should (see next paragraph), this option can be enabled. **

### Authentication

The inspector app needs authentication for two reasons:

1. Preventing unauthorized changes to the data
2. Knowing which users made which changes to the data

Currently, authentication is implemented with the package [{shinymanager}](https://github.com/datastorm-open/shinymanager). It reads the usernames form the configuration items and uses a standard password for all the accounts. In development phase this is accepatble, but in production the user database should - at least - be implemented with for example [{RSQLite}](https://github.com/r-dbi/RSQLite). The package documenation gives directions ho to implement sucg a database.

Depending the way you wish the  app to be deployed, you could also decide to use a proxy. If you want to deploy the app as containers in the cloud, you could, for example, consider [{shinyproxy}](https://github.com/openanalytics/shinyproxy).&#x20;

## Structure of Package

The idea of app-as-an-package is borrowed from the {golem} package. However, the methodology was followed only to a limited extent. Only in the following cases {golem} has been used:

* Script run\_inspector.R  to start the inspector app
* Code for starting the inspector app with arguments
* Code for adding external resources (e.g. js libraries)
* Generating app.R script to deploy the app  on shinyapps.io
* Generating Dockerfile to deploy app in Docker Engine

```
// in repo and in package
.
├── CODE_OF_CONDUCT.md
├── DESCRIPTION
├── LICENSE
├── LICENSE.md
├── NAMESPACE
├── README.md
├── R
│   ├── app_main.R ......... sources for UI and server of the app
│   ├── config_template.R .. function to make a configuration template
│   ├── configuration.R .... documentation of the configuration items│
├── isoinspector.R ......... documentation of the package
│   ├── main_utils.R ....... utility functions of 'main'
│   ├── main_utils_InspectorObj_R6.R ... definition of R6 object
│   ├── main_utils_login.R .. specific utility of 'main'
│   ├── mod_edit.R .......... sources of module 'X'
│   ├── mod_edit_utils.R .... utility functions of module 'X'
│   ├── mod_filter.R
│   ├── mod_filter_utils.R
│   ├── mod_inspect.R
│   ├── mod_panel.R
│   ├── mod_panel_utils.R
│   ├── mod_plot.R
│   ├── mod_plot_utils.R
│   ├── mod_read.R
│   ├── mod_read_utils.R
│   ├── run_inspector.R ..... script to run the app
│   └── sysdata.rda ......... internal configurations
├── data .................... standard R package dir; not used
├── inst
│   ├── app
│   │   └── www
│   │       └── favicon.ico
│   ├── extdata .............. standard R package dir; not used
│   └── golem-config.yml ..... {golem} 
├── man
│   ├── InspectorObj.Rd ...... docs explaining the functions and objects
│   ├── ................
│   ├── ................
|   .
|   .
// in repo only
├── data-raw
│   ├── app_config.R ......... script to add configurations to package
│   └── clumped_isotope.json . configurations
├── inspector_manifest.txt ... list of files to include in app
.
//generated locally
.
├── app.R ..................... script to deploy app on server
├── Dockerfile ................ Prescription for making a Docker image
```

#### &#x20;Package and/or repository

As alreay mentioned in the [Preliminaries](inspector-app-deployment-guide.md#preliminaries) there is a distinction between an package (R) and a repository (GitHub). There is extensive documentation on how the two interplay. See for instance the book [R Packages](https://r-pkgs.org/index.html). Make sure you understand the role of each before deploying the inspector app for your research group.

#### Documentation

The modules, functions and objects are documented inside the package. These descriptions are available by means of the help function in R/RStudio. Each document is separately accessible, but you can walk down the hierarchy of functions starting with:&#x20;

```
library(isoinspector)
?isoinspector
```

#### Configuration

&#x20;Directory `data-raw` contains scripts and json files for making configurations as [internal data](https://r-pkgs.org/data.html#data-sysdata). As such it belongs to the repository (is under git versioning), but doesn't belong to the package. For more on how to make configurations see [here](inspector-app-deployment-guide.md#how-to-configure-an-inspector-app).

#### Deployment

The files `app.R` and `Dockerfile` are generated by tools in the package [{golem}](https://github.com/ThinkR-open/golem). They are used in different deployment settings. See [next chapter](inspector-app-deployment-guide.md#how-to-deploy-an-inspector-app) for more info.

## How to deploy an inspector app?

### On a local workstation

After installing the package, you can run the app by issuing:

```
library(isoinspector)
run_inspector(config = "configX")
```

Where configX is a build-in configuration in the package or a file `configX.json` in the workdir of the app. How to build your own configuration is explained in [this chapter of the document](inspector-app-deployment-guide.md#how-to-configure-an-inspector-app).

### On the cloud service `shinyapps.io` of Rstudio

To gain access to shinyapps.io see [Prerequisites](inspector-app-deployment-guide.md#prerequisites).&#x20;

To deploy an Inspector app on a service, you need to have git cloned the repo from your fork. Go to the start directory of the repo and run:

```
library(golem)
add_shinyappsio_file()
```

This will produce the file app.R. Open this file with an editor (RStudio editor will do) and change the call:

> run\_app()&#x20;

into

> run\_inspector(config = 'configX')

Where configX is one of the build-in configurations. It's not possible to use configuration files in the working directory, because in shinyapps.io you have little control over the working directory.

Then deploy the app on shinyapps.io:

```
library(rsconnect)
deployAppdeployApp(
  appFileManifest = 'inspector_manifest.txt', 
  appName = 'your-app-name', 
  account = 'your-account-name')
```

The file `manifest.txt` specifies which files and directories must be uploaded to shinyapps.io. After (quite) a while the app will start up in a browser window. TIP: add the app address to your bookmarks.&#x20;

### In a container on a `Docker Engine`

Assuming you have a Docker Engine installed and running on your local workstation (see Prerequisites), the following shows how you get an inspector app running in a container.

Make a Dockerfile by

```
library(golem)
add_docker_file()
```

Edit the Dockerfile (e.g. in your RStudio Editor pane) and change the line

> CMD R -e "options('shiny.port'=80,shiny.host='0.0.0.0');isoinspector::run\_app()"

into

> CMD R -e "options('shiny.port'=80,shiny.host='0.0.0.0');isoinspector::run\_inspector(config = 'myconfig')"

Then use docker commands in the terminal of your local workstation to make a container:

> docker build -t my\_inspector .      # don't forget the dot at the end

While your machine is building an docker image you can take  a long coffeebreak or you can do your daily exercises.&#x20;

And then start the container

> docker run -dp 127.0.0.1:80:80/tcp  my\_inspector

The app can be viewed by pointing your browser at 127.0.0.1:80. In this situation the container can't read/write directly from/to the file system of your local machine. But if you that's what you want (for using local configurations or storing output and recovery files), you can mount a directory to the working directory of the container

> docker run -dp 127.0.0.1:80:80/tcp -v /path/folder:/buildzone my\_inspector

## How to configure an inspector app?

As administrator or as user you can create a new app by installing a new configuration or adjusting an old one. This can be accomplished in two ways:

1. Place a new/adjusted configuration file (.json) in the working directory of the app. This is typically useful for users who run the app on their local workstation.
2. Repackage the app with the new configuration added to the internal data. Typically meant for apps that are deployed on servers (e.g. shinyapp.io) or are containerized (Docker).

For both strategies the best way to make a new configuration is to use an existing one as template. You can produce such a template with

```
library(isoinspector)
config_template(config_name = 'an_existing_configuration')
```

The output is a file named `config_template.json`. Edit the file and change the filename to the name of your new configuration. The definitions of the various elements of a configuration can be found in the on-line help.

```
library(isoinspector)
?configuration
```

For local use, copy the adjusted file to the working directory of your app or its subfolder `inspector.`

For deployment on a server and/or in a Docker container its better to add the new configuration to the internal data the package. This assumes that you have fork-cloned the repo of the package.  Place the new configuration json file in the subdirectory `data-raw` and edit the script app\_config.R in the same directory and execute it in the home directory of the repo. See also chapter 14.2 of the book [R Packages](https://r-pkgs.org/index.html) for how to install internal data inside a package.

In both situations: run the new app with `run_inspector(config = 'myconfig')`. Where `myconfig` is the name of the configuration data.&#x20;





