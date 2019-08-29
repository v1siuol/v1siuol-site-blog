# Flask Traditional Deployments: Zero to One &lt;Deploy uWSGI&gt;

> The following article wil introduce how to install a production-ready web server.

> Author: v1siuol
>
> Last-Modified: 2018.08.14

### Background 

- A quite developed web project ( For Flask case, `flask run` listens *127.0.0.1:5000* by default. However, `flask run --host 0.0.0.0` listens *a.b.c.d:5000*  under the same wifi. Have fun with it first, then come and playing with VPS! )
- This series is a supplyment for *Tradition Deploments* in *Chapter 17. Deployment* in *[Flask Web Development, 2nd Edition][url_flask_web_dev]* written by *Miguel Grinberg*. 
- Project address: https://github.com/v1siuol/v1siuol-site



### 0. Try uWSGI 

Official documents: https://uwsgi-docs.readthedocs.io/en/latest/WSGIquickstart.html 

```bash
$ uwsgi --http 127.0.0.1:3031 --wsgi-file v1siuol.py --callable app --processes 4 --threads 2 --stats 127.0.0.1:9191
```

We choose `--http` instead of `--socket` since we will use `uwsgitop` later to determine how many processes and thread  are *optimal*. 

> There is no magic rule for setting the number of processes or threads to use. It is very much application and system dependent. Simple math like `processes = 2 * cpucores` will not be enough. You need to experiment with various setups and be prepared to constantly monitor your apps. `uwsgitop` could be a great tool to find the best values.
>
> Reference: https://uwsgi-docs.readthedocs.io/en/latest/ThingsToKnow.html

Running the above line, we are expecting to see something like:

![uwsgi_running](https://www.v1siuol.com/files/uwsgi_running.png)

### 1. Config uWSGI 

Keep uWSGI running, we now open another terminal, logging in the server via ssh. 

```bash
# After login, new ssh connection
$ cd flask/
$ cd v1siuol-site/
$ source venv/bin/activate
(venv)$ uwsgitop :9191
```

The port uwsgitop listen to should be the same as the port uWSGI put the stat in. 

![uwsgtop_start_moniter](https://www.v1siuol.com/files/uwsgtop_start_moniter.png)

Since uwsgi and uwsgitop are ocupying two terminals, we open the third terminal to simulate the regular access to our flask website. 

```bash
# After login, another new ssh connection
$ curl -v localhost:3031  # You may repeat it, but this only accesses index page
```

Meanwhile, CHECK your uwsgitop window. 

![uwsgitop_on](https://www.v1siuol.com/files/uwsgitop_on.png)

See! Here we moniter the reaction under 4 processes, each with 2 threads. Obviously, we don't need that much (though we only consider one connection cases). 



Try a few times and find the best values. THEN we shall move to the next step, configing uWSGI in a more proper way. 

```bash
(venv)$ sudo mkdir /etc/uwsgi
(venv)$ sudo chmod 777 /etc/uwsgi 
(venv)$ touch /etc/uwsgi/emperor.ini
(venv)$ vim /etc/uwsgi/emperor.ini
```

#### > [ emperor.ini ]

In `emperor.ini` :

```bash
[uwsgi]
emperor = /etc/uwsgi/vassals
uid = 1000
gid = 1000
```



```bash
(venv)$ sudo touch /etc/systemd/system/uwsgi_v1siuol.service 
(venv)$ chmod 777  /etc/systemd/system/uwsgi_v1siuol.service
(venv)$ vim /etc/systemd/system/uwsgi_v1siuol.service
```

#### > [ uwsgi_v1siuol.service ]

In `uwsgi_v1siuol.service` :

```bash
[Unit]
Description=uWSGI instance to serve flask
After=syslog.target

[Service]
ExecStart=/home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini /etc/uwsgi/emperor.ini
# Requires systemd version 211 or newer
RuntimeDirectory=uwsgi
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```



```bash
(venv)$ sudo systemctl start uwsgi_v1siuol.service
(venv)$ systemctl status uwsgi_v1siuol.service  # check if active
```

We are almost there, but one vassal left. 

```bash
(venv)$ sudo systemctl stop uwsgi_v1siuol.service
(venv)$ touch /etc/uwsgi/vassals/uwsgiconfig.ini
```

#### > [ uwsgiconfig.ini ]

In `uwsgiconfig.ini` :

```bash
[uwsgi]

chdir = /home/ubuntu/flask/v1siuol-site

socket = /home/ubuntu/flask/v1siuol-site/logs/uwsgi.sock
chmod-socket = 666
stats = /home/ubuntu/flask/v1siuol-site/logs/uwsgi_stat.sock

wsgi-file = v1siuol.py
callable = app

logto = /home/ubuntu/flask/v1siuol-site/logs/uwsgi.log
```

Try again!

```bash
(venv)$ sudo systemctl start uwsgi_v1siuol.service
(venv)$ ps aux | grep uwsgi
```

> ubuntu   23216  0.0  0.4  34696  4252 ?        Ss   13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini /etc/uwsgi/emperor.ini
>
> ubuntu   23218  0.4  6.1 144172 62620 ?        S    13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23222  0.0  5.3 144172 54456 ?        S    13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23236  0.0  0.1  12916  1084 pts/1    S+   13:55   0:00 grep --color=auto uwsgi

Great! We finally daemonize uWSGI. BUT plz never ever daemonize the Emperor unless you know what you are doing!!!  

### 2. Kill uWSGI

Now you can do something really fun. 

```bash
(venv)$ ps aux | grep uwsgi
```

> ubuntu   23216  0.0  0.4  34696  4252 ?        Ss   13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini /etc/uwsgi/emperor.ini
>
> ubuntu   23218  0.0  6.1 144172 62620 ?        S    13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23222  0.0  5.3 144172 54456 ?        S    13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23368  0.0  0.0  12916   932 pts/1    S+   15:31   0:00 grep --color=auto uwsgi

```bash
(venv)$ kill -INT 23222
(venv)$ ps aux | grep uwsgi
```

> ubuntu   23216  0.0  0.4  34696  4252 ?        Ss   13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini /etc/uwsgi/emperor.ini
>
> ubuntu   23218  0.0  6.1 144172 62620 ?        S    13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23382  0.0  5.3 144172 54456 ?        S    15:40   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23384  0.0  0.1  12916  1016 pts/1    S+   15:40   0:00 grep --color=auto uwsgi

Can you tell the difference of the **starting time** and **pid** between processes? 

pid: 23382 is reloaded!!!

Try to kill another one.

```bash
(venv)$ kill -INT 23218
(venv)$ ps aux | grep uwsgi
```

> ubuntu   23216  0.0  0.4  34664  4264 ?        Ss   13:53   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini /etc/uwsgi/emperor.ini
>
> ubuntu   23389 65.0  6.1 144188 62760 ?        S    15:40   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23392  0.0  5.3 144188 54372 ?        S    15:40   0:00 /home/ubuntu/flask/v1siuol-site/venv/bin/uwsgi --ini uwsgiconfig.ini
>
> ubuntu   23394  0.0  0.1  12916  1088 pts/1    S+   15:40   0:00 grep --color=auto uwsgi

Surprise! They all being reloaded! 



### 3. Conclusion

For editing files reference: 

`/etc/uwsgi/emperor.ini` 

`/etc/uwsgi/vassals/uwsgiconfig.ini` 

`/etc/systemd/system/uwsgi_v1siuol.service`  

For log files reference:

`/home/ubuntu/flask/v1siuol-site/logs/uwsgi.log`

`/home/ubuntu/flask/v1siuol-site/logs/uwsgi.sock` 

`/home/ubuntu/flask/v1siuol-site/logs/uwsgi_stat.sock`



Thank you. 



References: 

- https://uwsgi-docs.readthedocs.io/en/latest/Systemd.html
- https://uwsgi-docs.readthedocs.io/en/latest/Emperor.html

