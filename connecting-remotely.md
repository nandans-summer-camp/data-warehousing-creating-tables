# Working with remote processes

This document will explain a couple of ways to work with remote servers and databases. The solution you choose will depend on the exact way your infrastructure is set up, so this is just meant to explore a couple of tricks:


# Connect to a remote database from your local machine

This allows you to work with local files (`.sql` files and `.py` files and a local jupyter notebook) and connect to a remote database running on a server somewhere in the world (for example, AWS). 

This is a common situation! Our databases are usually running on a server somewhere. 

To do this, we need two things:

1. We can make network connections (think HTTP requests) from our local machine to the database server. This either means that the database is available on the public internet or that we use SSH port forwarding.

2. We have a local database client software that can communicate with the database.


### Connecting over the internet

To do this, we need to expose our server to the internet. Using AWS EC2, this means opening up the relevant port (for example 5432) with the relevant protocol (for Postgres, TCP) in the security group attached to the instance. 

Once it's exposed to the internet, anyone can connect to it, including us! This means that you probably want to: 

1. Run your Postgres server with a password. 

And optionally: 

2. (Advanced) Run Postgres with SSL (encryption) so your data is encrypted as it moves over the internet. 

3. (Annoying if not on "work internet" with fixed IP) Restrict the IP addresses that can access the server.

### SSH port forwarding

SSH port forwarding allows us to use the secure (encrypted) SSH connection to connect to a database (or server of any kind) running on a remote server.


``` shell
ssh -i mykey.pem -L 5432:localhost:5444 [USER]@[DNS]
```

Where `[DNS]` is the **Public DNS** of your AWS EC2 instance (you can also replace this with the **Public IP** address of the instance), `[USER]` might be `ubuntu` if you launched an Ubuntu instance on AWS, and `mykey.pem` is your local path to the private key file of the keypair used to launch the instance.

This allows you to connect to `locahost:5432` on your local machine and your connection will be "forwarded" to `localhost:5444` on the remote machine.

We can also use SSH port forwarding to securely connect to our Jupyter Notebook, without needing to expose `8888` via the firewall (security group) in AWS:

``` shell
ssh -i mykey.pem -L 8888:localhost:8888 [USER]@[DNS]
```

This is an easy way to have an encrypted connection to your remote notebook and avoid messing with security groups at the same time!


### Install a local client

You can now connect to the database (Postgres) server running on your EC2 instance FROM your local computer!

But, of course, you will need to install a client on your local computer.

Options:

1. psql in Postgres docker image, as we have been using.
2. psql - you could just install it locally on your machine instead of using it inside Docker.
3. pgadmin.
4. dbeaver.
5. Postgres extension in VSCode.
