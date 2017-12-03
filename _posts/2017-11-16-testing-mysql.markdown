---
layout: post
title:  "Testing a MySQL function"
categories: python mysql tests TDD
author: Francesco Montesano
---

## A simple MyQSL query

For a project at work I had to add a simple function to get a number from a
[MySQL](https://www.mysql.com/) table, increase it, add it back and return the
number. This is more or less it:

```python
import pymysql

def get_obsnumber(conf):
    '''Get a number from the database'''
    conn = pymysql.connect(host=conf['host'], port=conf['port'],
                           user=conf['user'], password=conf['password'],
                           database=conf['database'])
    cursor = conn.cursor()
    # get the number
    cursor.execute('SELECT MAX(obsnum) FROM my_table')
    obsnumber = cursor.fetchone()[0]
    # increase it
    obsnumber += 1
    # set it back
    cursor.execute('INSERT INTO my_table (obsnum) values (%s)', obsnumber)
    conn.commit()
    # close everything and commit
    cursor.close()
    conn.close()

    return obsnumber
```

I was given an example of the code, so I knew what I had to implement, but I had
to figure out how to test it.

## How hard can it be?

For the tests I needed to start a MySQL server, add the ``my_table`` table and
some row and they throw away the server at the end.

Sounds easy, doesn't it?

Unfortunately I never used MySQL before, so I started searching online for
tutorials and solutions. 

### ``pytest-mysql``

At first I decided to give
[pytest-mysql](https://github.com/ClearcodeHQ/pytest-mysql) a try. From the
documentation it seemed simple enough to use and, being a
[pytest](https://docs.pytest.org) plugins, it integrates well with my test
suite.

On my work computer 
I started to test it on my work computer, a Kubuntu 16.04 box, with MySQL 5.7.
At first I had to install a few packages.

OpenSUSE cannot start the server, because the use is set to ``None``, no matter
what. On Kubuntu doesn't work because ``mysql_install_db`` [has been
deprecated](https://dev.mysql.com/doc/refman/5.7/en/mysql-install-db.html) in
favour of ``mysqld --initialize/--initialize-insecure`` and many packages has
not yet been updated.

### Systems 

* OpenSUSE Tumbleweed: mysql  Ver 15.1 Distrib 10.1.25-MariaDB, for Linux
* (x86_64) using readline 5.1
* (K)ubuntu 16.04:

### Software that helps

* [my_virtualenv](https://github.com/evgeni/my_virtualenv): bash script, works
  on OpenSUSE, fails on Kubuntu

## The docker way

[example of docker mysql](https://severalnines.com/blog/mysql-docker-containers-understanding-basics)
[example of docker compose](https://github.com/bossbossk20/docker-compose-mysql/blob/master/docker-compose.yml)
