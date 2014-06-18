Bowie
=====

__STATUS: speculative; for conversation; not yet implemented; specification for a minimal viable product.__
    
Keep on publishing. Keep on re-releasing. Keep on playing.

Description
===========

An application manager come application runner. Why? To keep multiple apps on multiple machines running latest builds without local intervention. For example an installation requiring multiple raspberry pis to run the same suite of applications. 

It runs using node.js. It heavily leverages [forever.js](https://github.com/nodejitsu/forever-monitor) for process management. Bowie applications in theory may be programmed in any language. 

It is intended for use on project specific, bespoke deployments. This influences decisions in design. It is probably best run on local networks or intranets. At least one bowie server is likely run for one project, not one bowie server for multiple projects. At least for a minimal viable product.

Bowie Collections
=================

A bowie collection is a series of application to be kept running and periodically auto updated. Collections are to be configured on multiple non-development machines.

Installing
----------

`npm install bowie -g`

Working Directory
-----------------

Bowie should be run from its own directory. 

Enter into an empty directory (the bowie directory to hold the collection of app) and run `bowie init --collection`. You will be prompted to add the url of the bowie repository server and your access token. When finished running it will set the state of the current directory to at least:


     /.
     /..
     /bowie.collection.json
     


Installing An App
-----------------
     
To add an app to bowie run `bowie install myapp` which sets the current directory to a state of at least:

     /.
     /..
     /bowie.collection.json
     /myapp/bowie.app.json
     
 
Starting A Bowie Collection
---------------------------

Enter the bowie directory and run `bowie startall`. This starts all applications. It will also periodically check for updates to any new application. This, for example, maybe set as as cron job for `@reboot`.

Stopping A Bowie Collection
---------------------------

Enter the bowie directory and run `bowie stopall`. This will stop all applications. It will also stop periodically checking of updated applications. This, for example, maybe used to temporarly halt a machine from autoupdating or restarting application so it becomes a development/test machine until next reboot or until `bowie startall` is ran again.

Bowie Apps
==========

Bowie apps are the apps which are run within collections. They can be published to the bowie repository. They will be created and published on, likely, a single development machines.  

Enter into the application/project working directory and run `bowie init --app`. This will create a `bowie.app.json` 


Creating 
--------

Enter the root directory of your application and run `bowie init --app`. This will create `bowie.app.json`. 

Configuring
-----------

The `bowie.app.json` has the following format:

    {
    	'name' : '',
    	'version' : '0.0.0',
    	'install_script' : {
    		'command' : name<String>,
    		'args' : arguments<Array>             // optional
    	},
    	'launch_script' : '{
    		'command' : name<String>,
    		'args' : arguments<Array>,            // optional
    		'forever_options' : options<Object>   // optional
    	},
    }
    
### name

This is the application name. It needs to unique to any bowie repository it is to be published to or the behaviour will be undefined.

### version

This is the version number of the application. Version numbers need to follow [Semantic Versioning](http://semver.org) and be parseable by [Semver](https://www.npmjs.org/package/semver). It is likely presumed that bowie apps will use the PATCH componenet of the version number to imply build number during development. Therefore bumping the PATCH number during the build process may be advisable.

### install_script

This is the script which is to be run by bowie when a new version of an application is first downloaded from a bowie server. It could, for example, install dependancies. It must adhere to the exit codes detailed below. Scripts which exit in failure may be reattempted, possibly only for a set number of times.

### launch_script

This is the script  is to be run by bowie and kept running. It should adhere to the exit codes detailed below. For example if the repeatedly exits with failure it may not be attempted to be restarted.

### forever_options

Bowie uses [forever.js](https://github.com/nodejitsu/forever-monitor) to keep process alive. The default forever option object of bowie can be extended with any object provided here. 

Testing/Preflight running
-------------------------

Not quite testing per sé but a util to preflight run the `install_script` and `launch_script` scripts. They can be run with `bowie run --install_script` or `bowie run --launch_script`. These don't do much other than run the commands and arguments specified in the `bowie.app.json`.

Exit codes
----------

Application should exit with code `0` for success and code `1` for failure. The behaviour of other exit codes is undefined.

Versioning
----------

The app version in `bowie.app.json` may be bumped by running `bowie bump:major`, `bowie bump:minor` or `bowie bump:patch`.

Publishing
----------

The app can be published by running `bowie publish`. When publishing you will be requested for the token which matches the `BOWIE_PUBLISHER_TOKEN_HASHES` hash provided as an enviroment variable to the bowie repository server.

 
Bowie Repository Server
=======================

A bowie repository server acts as a remote application repository to collections.

Running a Repository Server
---------------------------

The bowie server resides within bowie's own source code repository. Bowie stores published application packages to the folder `./application_packages/`. This is ignored by the bowies own source code git repository. This folder could in theory be a git repository provided it keeps an empty file named `.gitkeep`.

The server is run using a series of require environment variables and then `node app.bowie-server.js`. For example:

     $ PORT=8888 BOWIE_PUBLISHER_TOKEN_HASHES=somehash,anotherhash BOWIE_CONSUMER_TOKEN_HASHES=somehash,anotherhash node app.bowie-server.js
     
The environment variables are described below.

### PORT

The port the server will run on.

### BOWIE_PUBLISHER_TOKEN_HASHES

The comma separated string containing the hashes of publish tokens. When trying to publish an app bowie will request the token via stdin (the command line). Hashes can be generated in node.js, see the section Generating Hashes.

### BOWIE_CONSUMER_TOKEN_HASHES

The comma separated string containing the hashes of consumer tokens. A collection will need a consumer token in `bowie.collection.json` to be able to update its application. Hashes can be generated in node.js, see the section Generating hashes.

Generating Hashes
-----------------

Hashes can be generated on the command line using node with the following example:
    
    $ node
    > crypto.createHash('sha1').update('newpass').digest('hex')
    '6c55803d6f1d7a177a0db3eb4b343b0d50f9c111'
    > [CTRL-D] 


Wishlist
========

* `bowie.collection.json` should also install apps so the file itself is only need to move the collection.
* Remote logging of installs, exits, starts etc. Possibly with MAC codes or other ice methods.
* Bowie checking itself for updates?
* Syncing server to provide bowie collections? This would allow more apps to be added to a collection remotely. Maybe over kill.
* A web interface for viewing builds, publish times and possibly installs.
* A hook system for when applications a published. This, for example, could commit and push changes in the `./application_packages/` directory on the server.
