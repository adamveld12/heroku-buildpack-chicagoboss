# Heroku buildpack: ChicagoBoss


This is a Heroku buildpack for applications written with the ChicagoBoss web framework (master branch or 0.9 when release).

To select the Chicago Boss version you want to deploy, select the correct revision in your rebar.config :

```erlang

{deps, [
  {boss, ".*", {git, "git://github.com/ChicagoBoss/ChicagoBoss.git", "master"}}
]}.

```

The buildpack will automatically provision a dev database and configure your application to use it.

### Gotchas

##### You're running cb_admin or other apps
Deploying with cb_admin wired up in the boss config will (for some reason I haven't determined yet) screw up the routing when its running on heroku. This means none of your routes will resolve correctly and they will error out. The boss config transformer automatically removes all addons except for your app. This means that using addons isn't officially support at the moment.

##### You're using the multi build pack
Make sure you set a PATH variable in your config vars to the values from bin/release PATH: in the bin/release file. Doing this will allow the scripts to see the Erlang VM binaries.

You will also want to add the postgresql addon (or mongoDB via mongoHQ) by hand since it seems like the release script doesn't run if you use multiple buildpacks. For postgres you will have to add a DATABASE_URL config var set to the connection string for your database instance.

### Configure your app

    $ heroku config:add BUILDPACK_URL="https://github.com/mithereal/heroku-buildpack-chicagoboss.git" -a YOUR_APP

or
    
    $ heroku create --buildpack "https://github.com/mithereal/heroku-buildpack-chicagoboss.git"

### Connecting to your postgresql database

    $ heroku pg:psql

### Select an Erlang version

The Erlang/OTP release version that will be used to build and run your application is now sourced from a dotfile called `.preferred_otp_version`. It needs to be the branch or tag name from the http://github.com/erlang/otp repository, and further, needs to be one of the versions that precompiled binaries are available for.

Currently supported OTP versions:

* master (R17B pre)
* master-pu (R16B pre)
* OTP_R15B
* OTP_R15B01
* OTP_R15B02
* OTP_R16B
* OTP_R16B01
* OTP_R16B02
* OTP_R16B03

To select the version for your app:
get the version of OTP you developed with:

    erl -eval 'erlang:display(erlang:system_info(otp_release)), halt().'  -noshell | sed "s/\"*//g"  > .preferred_otp_version
    $ git commit "Select XXX as preferred OTP version" .preferred_otp_version

### Build your Heroku App

    $ git push heroku master

You may need to write a new commit and push if your code was already up to date.

### Datastores

This buildpack allows the use of the in-memory mock database, mongoDB and postgres by detecting the following keywords used in the boss.config: 

Database  | Keyword | Env Var
--------- | ------- | -------
 ETS/Mock | mock    | N/A
 Mongo DB | mongodb | MONGOHQ_URL
 postgres | pgsql   | DATABASE_URL
 
If you are using the mock database, be aware of the [memory constraints of heroku dynos and the consequences](https://devcenter.heroku.com/articles/dynos#memory-behavior)

IMPORTANT: All file uploads go to the transient file system, meaning that everything is destroyed on each deploys. These assets should go to S3 or similar.

Pull requests are very welcome.

### LIMITATIONS

Given the architecture of Heroku, it is not advisable to use the MQ with one or more dynos. They are isolated and each would have their own queue.

### THANKS

Geoff Cant for writing the base heroku-buildpack-erlang, the starting point of this buildpack. And the Chicago Boss and Heroku guys.

### LINKS

Chicago Boss: http://www.chicagoboss.org

Heroku: http://heroku.com

### AUTHOR

Eric Cestari

http://twitter.com/cstar

http://eric.cestari.info/
