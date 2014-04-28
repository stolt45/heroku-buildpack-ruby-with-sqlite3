Heroku buildpack: Ruby with SQLite3
===================================

This is a fork repository from [heroku-buildpack-ruby](https://github.com/heroku/heroku-buildpack-ruby)
and it is revised in order to use SQLite3 on Heroku.


Heroku restricts using SQLite3
-----
Heroku does not recommend using SQLite3 on the system; for more information, 
please refer Heroku's document page [SQLite on Heroku](https://devcenter.heroku.com/articles/sqlite3)
and read it carefully.
The reason is that Heroku's Cedar stack has an ephemeral filesystem.
This means that SQLite3's database contents will be cleared periodically.
Therefore Heroku deliberately restricts the SQLite3 deploy.


Why we still need SQLite3 on Heroku
-----
As you know well, SQLite3 can manage all kind of database on only one data file.
It is very easy to handle the data and backup it.

On Heroku's ephemeral filesystem, SQLite3 dose not fit every purpose, 
but in certain applications, it is more useful rather than PostgreSQL database.
It is the case of less frequently to update the database, besides 
the data is updated by only the site owner, not site visitor.



Proper application example
-----
* ZIP code and city name database
* Monthly average temperature and precipitation record for past 100 years
* Blog that is updated by only one site owner


Not proper application example
-----
* application that the database will be updated frequently by site visitor; like BBS system.



Usage
-----
First, please develop your App and SQLite3 database file on your local site.
Next, store your App and SQLite3 database file together into Git,
and deploy (git push) it to Heroku.

The detail is like following


### Gemfile
Please add "sqlite3" to Gemfile file and run "bundle install"

    gem "sqlite3"



### config/database.yml
Specify sqlite3 for both sections of **development:** and **production:**.

    # SQLite3 configuration

    development:
      adapter: sqlite3
      database: mydata.sqlite3

    production:
      adapter: sqlite3
      database: mydata.sqlite3



  
### BUILDPACK_URL
Before deploy (git push), set environment variable BUILDPACK_URL like following.
Please refer [Using a custom Buildpack](https://devcenter.heroku.com/articles/buildpacks#using-a-custom-buildpack)

```sh
$ heroku config:set BUILDPACK_URL=https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3
```



### Git and Deploy

```sh
$ git init
$ git add .
$ git commit -m "init"
$ git push heroku master
```

