# Flask Traditional Deployments: Zero to One &lt;Flask Server&gt;

> The following article wil introduce how to set up your first *flask server* after logging in your server, and some basic **linux**, **git** commands. 

> Author: v1siuol
>
> Last-Modified: 2018.08.14

### Background 

- A quite developed web project ( For Flask case, `flask run` listens *127.0.0.1:5000* by default. However, `flask run --host 0.0.0.0` listens *a.b.c.d:5000*  under the same wifi. Have fun with it first, then come and playing with VPS! )
- This series is a supplyment for *Tradition Deploments* in *Chapter 17. Deployment* in *[Flask Web Development, 2nd Edition][url_flask_web_dev]* written by *Miguel Grinberg*. 
- Project address: https://github.com/v1siuol/v1siuol-site



### 0. First Impression on Ubuntu 

![ubuntu_login](https://www.v1siuol.com/files/ubuntu_login.png)

Cong! You successfully login your virtual system.



But just after that, following warning is expected to pop up: 

> WARNING! Your environment specifies an invalid locale.
>
>  The unknown environment variables are:
>
>    LC_CTYPE=UTF-8 LC_ALL=
>
>  This can affect your user experience significantly, including the
>
>  ability to manage packages. You may install the locales by running:
>
>    sudo apt-get install language-pack-UTF-8
>
> ​     or
>
>    sudo locale-gen UTF-8
>
> To see all available language packs, run:
>
>    apt-cache search "^language-pack-[a-z][a-z]$"
>
> To disable this message for all users, run:
>
>    sudo touch /var/lib/cloud/instance/locale-check.skip

Solution:

```bash
$ export LC_ALL="en_US.UTF-8"
$ export LC_CTYPE="en_US.UTF-8"
$ sudo dpkg-reconfigure locales
```

Please do it, or you might encounter some weird issues below, like reporting 

> The virtual environment was not created successfully because ensurepip is not
>
> available.  On Debian/Ubuntu systems, you need to install the python3-venv
>
> package using the following command.

even though you might have already install `python3-venv`. 

Now try some commands like:

- `ls` : list directory contents 

- `pwd` : return working directory name 

- `man ls` : used to see `ls` manual pages

- `cd` : change current directory

- `mkdir` : make directories 

- `rm` : remove directory entries (be very very careful here)

  Tips: we can press `Tab` to automatically fillup the file name. 



### 1. Pull Your Project Down to Your Server

```bash
$ cd ~  # To my main directory 
$ mkdir flask  # to make a folder for my project 
$ cd flask/ 
$ sudo apt-get install python3-venv 
```

> Issue: you might fail to install, reporting: 
>
> > E: Package 'python3-venv' has no installation candidate
>
> Solution:
>
> `$ sudo apt-get update `
>
> Then install it again! 

```bash
$ sudo apt-get install python3-pip 
$ git clone https://github.com/v1siuol/v1siuol-site.git 
$ cd v1siuol-site/ 
$ python3 -m venv venv 
$ pip install --upgrade pip
$ source venv/bin/activate
(venv)$ pip install -r requirements.txt 
(venv)$ mv templates.env .env 
(venv)$ vim .env  # Complete the config 
(venv)$ deactivate  # Exit the venv
```

Now the flask project is almost ready to go. But one more thing left behind: **Setup the database**. 

### 2. MySQL on Ubuntu

```bash
$ sudo apt-get update
$ sudo apt-get install mysql-server
$ mysql_secure_installation
```

You need to do some selections in the last step and MySQL should be installed successfully in your server. Check by:

```bash
$ systemctl status mysql.service
```

> ● mysql.service - MySQL Community Server
>
>    Loaded: loaded (/lib/systemd/system/mysql.service; enabled; vendor preset: enabled)
>
>    Active: active (running) since Tue 2018-08-14 07:01:00 UTC; 8min ago
>
>  Main PID: 15973 (mysqld)
>
>    CGroup: /system.slice/mysql.service
>
> ​                  └─15973 /usr/sbin/mysqld
>
> Aug 14 07:00:59 ip-172-31-16-238 systemd[1]: Starting MySQL Community Server...
>
> Aug 14 07:01:00 ip-172-31-16-238 systemd[1]: Started MySQL Community Server.



Next, we will create the database for my flask project. 

``` bash
$ mysql -u root -p  # Enter the password you set a few minutes ago
```

> Welcome to the MySQL monitor.  Commands end with ; or \g.
>
> Your MySQL connection id is 6
>
> Server version: 5.7.23-0ubuntu0.16.04.1 (Ubuntu)
>
> Copyright (c) 2000, 2018, Oracle and/or its affiliates. All rights reserved.
>
> Oracle is a registered trademark of Oracle Corporation and/or its
>
> affiliates. Other names may be trademarks of their respective
>
> owners.
>
> Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```mysql
(mysql) > SHOW databases;
(mysql) > CREATE database IF NOT EXISTS v1siuol_site DEFAULT CHARSET utf8;
(mysql) > QUIT;
```

### 3. Do something Fun

```bash
(venv)$ flask run
```

> Traceback (most recent call last):
>
>   File "/home/ubuntu/flask/v1siuol-site/venv/bin/flask", line 11, in <module>
>
> ​    sys.exit(main())
>
>   File "/home/ubuntu/flask/v1siuol-site/venv/lib/python3.5/site-packages/flask/cli.py", line 894, in main
>
> ​    cli.main(args=args, prog_name=name)
>
>   File "/home/ubuntu/flask/v1siuol-site/venv/lib/python3.5/site-packages/flask/cli.py", line 557, in main
>
> ​    return super(FlaskGroup, self).main(*args, **kwargs)
>
>   File "/home/ubuntu/flask/v1siuol-site/venv/lib/python3.5/site-packages/click/core.py", line 676, in main
>
> ​    _verify_python3_env()
>
>   File "/home/ubuntu/flask/v1siuol-site/venv/lib/python3.5/site-packages/click/_unicodefun.py", line 118, in _verify_python3_env
>
> ​    'for mitigation steps.' + extra)
>
> RuntimeError: Click will abort further execution because Python 3 was configured to use ASCII as encoding for the environment.  Consult http://click.pocoo.org/python3/for mitigation steps.
>
> This system supports the C.UTF-8 locale which is recommended.
>
> You might be able to resolve your issue by exporting the
>
> following environment variables:
>
>   export LC_ALL=C.UTF-8
>
>   export LANG=C.UTF-8
>
> Click discovered that you exported a UTF-8 locale
>
> but the locale system could not pick up from it because
>
> it does not exist.  The exported locale is "en_US.UTF-8" but it
>
> is not supported

Sign :( So we add `export LC_ALL=C.UTF-8` `export LANG=C.UTF-8` to `.env` .

```bash
(venv)$ flask deploy
(venv)$ flask run --host 0.0.0.0
```

>  * Serving Flask app "v1siuol.py"
>
>  * Environment: production
>
>    WARNING: Do not use the development server in a production environment.
>
>    Use a production WSGI server instead.
>
>  * Debug mode: off
>
>  * Running on http://0.0.0.0:5000/ (Press CTRL+C to quit)

NOW turn up your brower, type your `<public IP>:5000` as url. 

> &lt;public IP&gt; can be found in your console:
>
> ![aws_public_ip](https://www.v1siuol.com/files/aws_public_ip.png)

WOW!!

![flask_own_server](https://www.v1siuol.com/files/flask_own_server.png)



Thank you. 