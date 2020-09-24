# PostgreSQL Tutorial

The purpose of this tutorial will be to walk you through the steps needed to run a relational database called PostgreSQL. 

PosgreSQL (or "Postgres" as it is often called) is a database that, while relatively modern, follows directly in the tradition of ACID, relational databases that use SQL as the language to write, mutate, and get data.

In databases which use SQL, the act of writing, mutating, and getting data is done by "making queries," where a "query" is nothing more than a peice of text written in the SQL query language. You will learn to write basic SQL to perform those data manipulations via the DataCamp classroom. In this tutorial, you will learn to run and set up the database. Much of the process of setting up the database is _also_ done in SQL queries, as we will see in this tutorial.

You can follow this tutorial on your local machine or on an AWS EC2 instance. 

## Running via Docker

Docker let's us run anything. This is why we love Docker! We don't need to install the Postgres database, we can just run it (which will first download the Docker image to our machine, as always):

`docker run --name postgres -p 5432:5432 postgres`

You should get an error. Try to follow it's instructions to fix the error! You can run it in the recommended way, with a password, or in the not recommended way, without a password. Both are fine for our purposes (as always, you should play around with both ways and make sure it all makes sense.)

You can run this process in the background by adding the `d` flag:


``` shell
docker run -d --name postgres -p 5432:5432 postgres
```

Good. If you now run `docker ps`, you should see that you have a postgres container running and listening on port 5432. 

So we have a database running on our machine. Now what? 

## Server + Client

Postgres, just most any databases, is built on a "client-server" architecture. Just like, for example, Jupyter Notebooks! 

The server does the heavy work of actually storing the data, writing it to disk, keeping it safe, etc. Each server can "serve" multiple (sometimes many!) clients at any given time. The clients make demands, they tell the server when they want it to write/mutate/get some data and the server follows their orders. 

The client communicates with the server in the SQL language. It sends SQL queries, the server fulfills the demands laid out in those queries, and sends a response back to the client. 

## Creating a client

So what we did previously was create the server. Now we need to create a client! Let's do that with ANOTHER container:

`docker run --rm --net host -it postgres psql postgresql://postgres@0.0.0.0`

You should now see a prompt that looks like this: 

``` shell
postgres=#
```

This is an interactive client. Similar to the Python interactive interpreter we accessed by running `python` (or `python3`), in some ways. You can exit the client with `ctrl+d`. After which, you can run the docker command again. 

You'll notice that when we ran the client docker continer we ran the image `postgres` with the command `psql psql postgresql://postgres@0.0.0.0`. `psql` is the interactive client we are running and you can learn about it here: https://www.postgresql.org/docs/12/app-psql.html.

We've connected with a user and (maybe) a password. We also told the client the IP address of the server (0.0.0.0), so it knows where to send its commands. We needed to specificy 0.0.0.0 along with `--net host` to get the docker container to contact our host (our machine) and send messages to the server running on our host (in particular, its running in another container, but forwarding communication to the host port 5432 as we set it up to do).

The default port for postgres is 5432, so we didn't need to tell it what port to communicate on, but we could've with `psql psql postgresql://postgres@0.0.0.0`. Similarly, we could've added the password to the connection string, if you set up your database with a password:  `psql psql postgresql://postgres:mypassword@0.0.0.0` (HINT: you will need this later if you setup your database with a password). This "connection string" is a common format for many databases and you will use it from within Python as well, for example. 


## Working with a client: SQL commands

OK, now we're going to perform some SQL commands. I will specify if the commands are postgres-specific or more generic "sql commands".

Each Postgres server can have multiple _databases_. To list the databases currently available in the server, you can use the postgres-specific `\l` command. Try that now. 

You should see that, in addition to a couple of "template" databases, which we won't worry about, you have one database: `postgres`.

Let's create a new database. We do this with a sql command: 

``` sql
CREATE DATABASE foo
```

You don't have to upcase the keywords in sql, but it is proper to do so for legibility whenever your sql will be read by someone!

OK, so we can now run `\l` again and see the database we created. To connect to a database we can ue the postgres-specific command: `\c nameofdatabase`. Try to connect to `foo`. You should see the prompt change to: 

``` shell
foo=#
```

We can describe the current database with `\d`:

``` shell
postgres=# \d
```

We can now create a _table_ in this database. Databases are made up of a set of _relations_, or tables. This is the core organizing principle of _relational databases_. Tables provide an abstraction for us to describe how we want our data to be stored (without actually caring how it's _literally_ stored on disk).

Tales have columns. Each column has a _data type_, such as `INT` for integer or `VARCHAR` for string types. Let's create a table with one column: 

``` sql
CREATE TABLE bar(id INT);
```

Now see what you have done with `\d`. You should see your table. If you want to describe the table, you can use `\d bar`. You can also view all the rows in the table bar by "selecting" all the data from bar: 

``` sql
SELECT * FROM bar;
```

`SELECT` is our basic "read" or "get" command for getting the server to send us the data in the database. Now let's write some data, which we do with `INSERT`:

``` sql
INSERT INTO bar VALUES(1);
```

Now let's select our data again:

``` sql
SELECT * FROM bar;
```

Congrats! You've A) created a database B) connected to the database C) created a table within that database D) written data to the table in the database and E) got data out of the table in he database. This is pretty much everything we need to do!

In the Datacamp course you will cover more about selecting data and writing sql queries to select data. For now, we just want to be able to create databases. 

Finally though, it can be tiresome to write everything in an interactive interpreter. Let's see how we can write, and then run, `.sql` files:

``` shell
nano foo.sql
```

You can use any text editor you wish, you don't have to use nano! Let's add the following lines:

``` sql
INSERT INTO bar VALUES(4);
SELECT * FROM bar;
```

Now let's run the query from the file (note that we're selecting which database we want to connect to by adding it to the end of the connection string. Previously, we connected to the default database "postgres", but now we want to connect to our newly created database "foo"):

``` shell
cat foo.sql | docker run -i --rm --net host postgres psql postgresql://postgres:mypassword@0.0.0.0/foo
```

What do you think will happen if you run that command multiple times? Make a guess, then try it. Does it make sense to you? 


## BONUS: Going Remote

Throughout this tutorial, I've assumed that we're running both the client and server on the same machine. But of course, one advantage of the client/server architecture is that they can run on different machines!

This will be very useful for your final project in this course, where you will need to have a database running in the cloud, so you might as well practice now!

Two options:

1. If you did everything locally, try to set up a Postgres server on an AWS EC2 instance and connect to it. 

2. If you did everything on an instance, try to set up a client locally and connect to the Postgres server running on your instance.

You can take a look at [connecting-remotely.md](connecting-remotely.md) for tips on setting up a work environment to connect remotely.
