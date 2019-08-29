# Ubuntu 时间魔法 &lt;Time Flies&gt;

> The following article will introduce how to set/change the timezone on Ubuntu via terminal.

> Author: v1siuol
>
> Last-Modified: 2019.08.02

![][]

### Background 

- Ubuntu 16.04



### 0. What time is it  

```bash
$ date
```

> Wed Aug 15 02:38:05 UTC 2018 



### 1. I got time magic 

```bash
$ sudo dpkg-reconfigure tzdata
```

Follow the directions in the terminal.

![][]

Geographic area ==>  Aisa

![][]

Time zone ==> one of (Chongqing, Hongkong, Macau, Taipei)

> Current default time zone: 'Asia/Chongqing'
>
> Local time is now:      Wed Aug 15 10:49:23 CST 2018.
>
> Universal Time is now:  Wed Aug 15 02:49:23 UTC 2018.



Also, you can check by `vim /etc/timezone` : 

> Asia/Chongqing 



Now you are all set! 





Thank you. 





Reference: 

- https://help.ubuntu.com/community/UbuntuTime

