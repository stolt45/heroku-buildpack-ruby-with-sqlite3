Heroku buildpack: Ruby with SQLite3
===================================

This is a fork repository from [heroku-buildpack-ruby](https://github.com/heroku/heroku-buildpack-ruby)
and it is revised in order to use SQLite3 on Heroku.

***
**!!! CAUTION !!!**  
**Please understand the RISK of the updated data vanishment on Heroku with SQLite3.**
***


Heroku restricts using SQLite3
-----
Heroku does not recommend using SQLite3 on the system; for more information, 
please refer Heroku's document page [SQLite on Heroku](https://devcenter.heroku.com/articles/sqlite3)
and read it carefully.
The reason is that Heroku's Cedar stack has an 
**[ephemeral filesystem](https://devcenter.heroku.com/articles/dynos#ephemeral-filesystem)**.
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

If you can prepare the data in advance,
you would update SQLite3 database by yourself locally and
store the application scripts and SQLite3 database file together,
then deploy (git push) it to Heroku.
It will work perfectly to read and query the database on Heroku.
(Of course, writing also works fine, but that data will be vanished soon.)
This is very easy operation.
And this is also effective to database scheme update.


Proper application example
-----
* ZIP code and city name database
* Monthly average temperature and precipitation record for past 100 years
* Blog that is updated by only one site owner (update blog locally and deploy it every time)


Not proper application example
-----
* application that the database will be updated frequently by site visitor; like BBS system.  


***


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
      database: db/mydata.sqlite3

    production:
      adapter: sqlite3
      database: db/mydata.sqlite3



  
### BUILDPACK_URL
Before deploy (git push), set environment variable **BUILDPACK_URL** like following.
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


***


Technical Explanation : Why it fails to bundle sqlite3 gem on Heroku
-----
On Ruby or Rails App, if you add "sqlite3" to Gemfile and deploy it
without custom buildpack (it means by default buildpack),
it will fail and not complete.

The reason is like following.


### Lack of **sqlite3.h** and **libsqlite3.so**
When sqlite3 gem is installed, it is making **native extension**.
This means that gem installer does **C compile** and **link**.
But Heroku's Cedar stack dose not have **sqlite3.h** that is included when C compiler is running,
and does not have symbolic link of **libsqlite3.so** library neither that is linked by linker.
In other words, **libsqlite3-dev** package is not installed on Heroku's Cedar stack.

In order to install sqlite3 gem properly,
we have to solve these two problems like **sqlite3.h** and **libsqlite3.so**.

In addition to that, Heroku has other two problems.


### Overwriting **config/database.yml** 
The default buildpack [heroku-buildpack-ruby](https://github.com/heroku/heroku-buildpack-ruby)
is overwriting **config/database.yml**  file and
sqlite3 configuration will be disappeared.


### Automatic installation of **heroku-postgresql:hobby-dev addon**
The default buildpack [heroku-buildpack-ruby](https://github.com/heroku/heroku-buildpack-ruby)
is automatically installing **heroku-postgresql:hobby-dev addon**.
This **heroku-postgresql addon** require the "pg" gem,
and it will cause the runtime error because "pg" gem is not installed.  


***


Rivised Points
-----



### Add sqlite3.h
I attached **sqlite3.h** file under **vendor** directory.

[vendor/sqlite3.h](https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3/blob/master/vendor/sqlite3.h)


### Symbolic link libsqlite3.so
For preparing libsqlite3.so, symbolic link from libsqlite3.so to /usr/lib/libsqlite3.so.0.8.6.

[lib/language_pack/ruby.rb](https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3/blob/master/lib/language_pack/ruby.rb)
```ruby
L525      run("ln -s /usr/lib/libsqlite3.so.0.8.6 #{yaml_lib}/libsqlite3.so")                        # for sqlite3   make symbolic link
```

### Copy sqlite3.h
For preparing sqlite3.h, copy from vendor/sqlite3.h to include directory.

[lib/language_pack/ruby.rb](https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3/blob/master/lib/language_pack/ruby.rb)
```ruby
L526      run("cp #{File.expand_path( "../../vendor/sqlite3.h", $PROGRAM_NAME )} #{yaml_include}")   # for sqlite3   prepare sqlite3.h
```



### Avoid overwiting to config/database.yml
config/database.yml overwiting method **create_database_yml** is called from L94
that is inside of  L81 **compile** method. 
In order to stop overwriting to config/database.yml, commented out this L94.

[lib/language_pack/ruby.rb](https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3/blob/master/lib/language_pack/ruby.rb)
```ruby
L94  #        create_database_yml        # for sqlite3    config/database.yml  should be kept intact
```


### Avoid heroku-postgresql:hobby-dev addon
In order to prevent from automatical installing **heroku-postgresql:hobby-dev addon**,
commented out these tree lines.

[lib/language_pack/rails2.rb](https://github.com/yotsumoto/heroku-buildpack-ruby-with-sqlite3/blob/master/lib/language_pack/rails2.rb)
```ruby
L65  #  def add_dev_database_addon                # for sqlite3   prevent from forcing addon 'heroku-postgresql:hobby-dev'
L66  #    ['heroku-postgresql:hobby-dev']
L67  #  end
```


***

