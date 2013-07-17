heroku-gss
==========

This repository contains a third-party [buildpack](https://devcenter.heroku.com/articles/third-party-buildpacks) that demonstrates that [GemStone/S 64 Bit](http://gemtalksystems.com/index.php/products/gemstones/) can be run in a [Heroku](https://www.heroku.com/) dyno. Of course, since each dyno gets an [ephemeral file system](https://devcenter.heroku.com/articles/dynos#the-dyno-manager), it is largely useless as a database or application server. It does, however, give you a way to run GemStone/S without doing your own install so you can look at and execute Smalltalk code in GemStone/S. 

If you aren't familiar with Heroku, check out the [quickstart](https://devcenter.heroku.com/articles/quickstart) instructions. Once you have completed steps 1-3, create a directory to be used for your GemStone/S application and initialize it as a git repository. For example:

    mkdir heroku-gss
    cd heroku-gss
    git init

Once there, you create an application. For GemStone/S we have a demo called [Webtools](https://github.com/jgfoster/webtools) that allows you to interact with GemStone/S through a browser. To use this buildpack, you must provide a file, `startup.tpz`, that starts your application. This file contains a script that will be run in Topaz:

    set user DataCurator pass swordfish gems gs64stone
    login
    input $GEMSTONE/examples/www/install.tpz
    run
    | server |
    server := GsSession currentSession userProfile symbolList last at: #'Server'.
    server startServerAtPort: (System gemEnvironmentVariable: 'PORT') asNumber.
    %
    exit

Note that we start the web server on a port defined by the $PORT environment variable. At this point we add our file to Git and commit our changes:

    git add startup.tpz
    git commit -m "first commit"

Now we tell Heroku about the application (and that it uses a third-party buildpack) and push it to the server:

    heroku create mygssdemo --buildpack https://github.com/jgfoster/heroku-gss
    git push heroku master
    heroku logs
    heroko open

You should select a unique name for your application (`mygssdemo` might be taken!). The application takes a few minutes to push and start. You can repeat the `heroku logs` command till you see 'State changed from starting to up' in the logs.

If anything strange happens, you can restart the application:

    heroku ps:restart
    heroku logs

When you are done, you should delete your dyno so that it does not take up resources:

    heroku apps:destroy mygssdemo

Note that we are running a special version of GemStone/S 64 Bit (3.1.0.5.alpha) that addresses a file-system issue on Heroku.

Have Fun!
