# CEDB is a secure web application for managing contacts, clean energy grants and community energy profiles

## About the Community Energy Database (CEDB) App

This app is demonstration tool and usability testing platform. It is used to work towards a full-featured group-accessible database for managing clean energy grant applications and community energy profiles. It provides a working tool for exploring user interfaces, data to be collected, and workflow functionality. It is also used to test data imports and highlight where data needs to be cleaned.

__Contacts__ - All the people, from Granters to Grantees.

__Energy Project__ - Tools to track grant applications made by communities for energy efficiency or to generate renewable energy.

__Community Energy Profiles__ - All information about a village/town/city, including facts like population and energy use, and links to Contacts, past and current Applications, and completed energy Projects.

## About the CEDB Code Framework and Security

### The Stack
CEDB is a [Drupal](https://drupal.org) version 8 application with a [PostgreSQL](https://www.postgresql.org/) database back-end running on top of a [Apache](https://httpd.apache.org/) web server. For code development, we are using [Lando](https://docs.devwithlando.io/) - a free, open source, cross-platform, local development environment and DevOps tool built on [Docker](https:docker.com/) container technology. Lando is developed by [Tandem](http://thinktandem.io/).

Using Lando we create a local dev environment that includes [Composer](https://getcomposer.org/), for managing library dependencies, and [Drush](https://www.drush.org/), for command line management of our Drupal site.

### Security

Don't ever add client data to Git. Ever.

Use CSV/feeds and SQL uploads to bring in test data.

The wonderful thing about Lando and Drush CEX (configuration export) is that CEX never captures the data in the database. It only captures structure. In order to capture the data, you will need to dump the database, for example using the Drupal [Backup and Migrate](https://www.drupal.org/project/backup_migrate) module. The key is to NEVER save your backup files to the folder that contains your CEDB code (and git repository).

## How to Install, Run and Improve CEDB

#### 0. Requirements ####

We are _developing_ CEDB using Lando, which is compatible with:

- MacOS 10.10+
- Windows 10 Pro+ (or equivalent) with Hyper-V* running
- Linux (with kernel version 4.x or higher)

* Hyper-V must be enabled on Windows. See [article](https://msdn.microsoft.com/en-us/virtualization/hyperv_on_windows/quick_start/walkthrough_install).

To _run_ an instance of CEDB, you will need a Linux server with PHP and Apache/NGINX available.

#### 1. Install Lando ####

Follow the instructions at https://docs.devwithlando.io/installation/installing.html

This nets you Docker and Composer.

#### 2. Create an empty directory in which to put CEDB ####

For example:

```
$ mkdir Users/yourName/code/sites/cedb
```

#### 3. Use git to pull the repo to this folder ####

E.g.
```
$ cd cedb
$ git clone git@github.com:cnburnett/cedb.git
```
Then, have a look at the Lando YML file.
```
$ more .landk.yml
```
You will note that CEDB was built to run connected to a PostgreSQL database server. You will need to change some settings to get CEDB to work (#5)

#### 4. Request information on the configuration from Lando ####
```
$ lando info
```

#### 5. Change Settings.php ####

Change the default Drupal _settings.php_ file so that Drupal connects to the PostgeSQL server (Docker container). Your _settings.php_ file is located here: /cedb/web/sites/default/

Here is what your settings configuration should look like:
```
$databases['default']['default'] = array (
  'database' => 'drupal8',
  'username' => 'drupal8',
  'password' => 'drupal8',
  'prefix' => '',
  'host' => 'database',
  'port' => '5432',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\pgsql',
  'driver' => 'pgsql',
);
```

#### 6. Launch CEDB ####
```
$ lando start
```

Lando will show you the local IP/domain:ports that you need to view CEDB in your browser
- Note, you can 'see' your app/site on another computer on the network using the IP and port (of the docker image), in my case http:192.168.0.YY:XXXXX - yay!

#### 7. Open CEDB in a browser ####

Here are some passwords:
- Drupal Username: admin
- Drupal Password: admin

## How to capture changes in Code (Github)

Using Drush to capture configuration (everything from Site Name to Block settings)
- The location in your site for configuration yml files is specified at the very end of settings.php file
- Build (export or update) config yml files using $ lando drush cex
- Then add to repo using $ git add . & git commit -am ""

## How to add a new Drupal Module

- Use composer (in lando)
- E.g. $ lando composer require drupal/feeds
- see docs at https://www.drupal.org/docs/develop/using-composer/using-composer-to-manage-drupal-site-dependencies
- Enable using:
- $ lando drush en feeds -y
- Check if a-ok:
- $ lando drush pm-list
- Change settings and configuration

## How to capture structural changes
- When you are happy, use cex to capture the changes and commit to git and Github

Build (export) the initial config yml files using:

```
$ lando drush cex
$ git status
$ git add * [IF NEEDED]
$ git commit -am 'YOUR MESSAGE'
```

## Update Modules and Drupal core

Read this: https://github.com/drupal-composer/drupal-project#updating-drupal-core as you may need to take care for scaffolding files (commonly .htaccess).

This may work just fine:

```
$ lando composer update
```

## Appendix: Initial CEDB Build Steps

References:

- Main Lando docs: https://docs.devwithlando.io/tutorials/drupal8.html#recipe
- A nice video tutorial: https://www.youtube.com/watch?v=JGef7Fx44F4

Step 1. Created code directory

```
$ cd /Users/yourName/code/sites/ (e.g.)
$ mkdir cedb
$ cd cedb
```

Step 2. Initialize the Lando project

```
$ lando init
```
- Answer some questions e.g.
  - What recipe? Select "drupal8"
  - Where root? Type in "web"
  - Name? Type "cedb"
- Lando runs the initiation and creates the Lando configuration file, "".lando.yml" in the empty directory.

Step 3. Start up the (empty) App

```
$ lando start
```
- Lando will pull in some tooling. In particular it pulls in Composer, Drush, Drupal console. Note that since no Drupal codebase existed at the time, the App does not work.

Step 4. Use composer to pull in the Drupal codebase via a template
- Note: We install into a empty directory and then move all directories/files up into the main director (e.g. pull template into 'blah' and the copy )
- Note: The composer call below grabs Drupal via a template stored in Github.
-- https://github.com/drupal-composer/drupal-project
- Note: We replace the some-dir from the copied call, with _blah_.  

```
$ lando composer create-project drupal-composer/drupal-project:8.x-dev blah --stability dev --no-interaction
```

Step 5. Copy files from the _blah_ folder one up...

```
$ mv blah/* .
$ mv blah/*.* .    // Move the dot files too
$ rmdir blah
```

Step 6. Init Git and add the .yml file

```
$ git init
$ git add -A
$ git commit -m 'Create basic Drupal 8 project with Lando'
```

Step 7. Swap out the recipe in the .lando.yml file, so we can have PostgreSQL

Start by stopping you Appendix

```
$ lando stop
```

Then, copy the template verbose recipe from the Lando documentation website: [here]https://docs.devwithlando.io/tutorials/drupal8.html#recipe

Then paste this into your .lando.yml file:

```
$ nano .lando.yml
```

Re-start your App

```
$ lando start
```

Step 8. Change settings.php so Drupal/PHP could connect to the PostgeSQL server. The settings.pgp file is located here: /sites/default/settings.php

```
$ sudo vim settings.php   # SHIFT-G to jump to bottom of file
```


```
$databases['default']['default'] = array (
  'database' => 'drupal8',
  'username' => 'drupal8',
  'password' => 'drupal8',
  'prefix' => '',
  'host' => 'database',
  'port' => '5432',
  'namespace' => 'Drupal\\Core\\Database\\Driver\\pgsql',
  'driver' => 'pgsql',
);
```

Step 9. Rebuild the App; request status from Lando; and retart Lando
```
$ lando rebuild
$ lando info
$ lando start
```
- Grab the local dev address

Step 8. Open the website dev address in browser; installed Drupal

Step 9. Add new config to Git

```
$ git init
$ git add *
$ git commit -am 'Adding revised recipe to use PostgresQL and Nginx'
```

Followed these instructions to set origin:

https://help.github.com/articles/adding-an-existing-project-to-github-using-the-command-line/

```
$ git remote add origin git@github.com:username/cedb.git
```

Step 10. Run installation

e.g.

Note that at Step 4. of the insatllation "Set Up the Database", in ADVANCED OPTIONS, change the host to 'database'

Step 11. After running the install, there will me a new directory... 'config/' that we need to add to Git.

Step 12. Start adding custom content types, views and blocks.

In the Drupal UI.
