---
layout: post
title:  "Testing a MySQL function"
categories: python mysql docker tests TDD
author: Francesco Montesano
---

## A simple MyQSL query

In one of my projects I had to add a function to get a number from a
[MySQL](https://www.mysql.com/) table, increase it, add it back and return the
number. This is more or less it:

```python
import pymysql

def get_obsnumber(conf):
    '''Get a number from the database'''
    conn = pymysql.connect(host=conf['host'],
                           port=conf['port'],
                           user=conf['user'],
                           password=conf['password'],
                           database=conf['database'])
    cursor = conn.cursor()
    # get the number
    cursor.execute('SELECT MAX(obsnum) FROM my_table')
    obsnumber = cursor.fetchone()[0]
    # increase it
    obsnumber += 1
    # set it back
    cursor.execute('INSERT INTO my_table (obsnum) values (%s)', obsnumber)
    # commit and close everything
    conn.commit()
    cursor.close()
    conn.close()

    return obsnumber
```

I was given an example of the code, so I knew what I had to implement, but first
I had to figure out how to test it.

## How hard can it be?

For the tests I needed to start a MySQL server, add the ``my_table`` table and
some row and then at the end throw away the server.

It sounds easy, doesn't it?

Unfortunately I never used MySQL before, so the first thing to do was to start
searching online for tutorials and possible solutions. 

## Create a temporary MySQL server.

At first I decided to give a go at
[pytest-mysql](https://github.com/ClearcodeHQ/pytest-mysql): it seems easy
enough to use and, being a [pytest](https://docs.pytest.org) plugin, would fit
into my workflow. The package provides a couple of fixtures: i) ``mysql_proc``
starts a MySQL server when first used and tears it down at the end of the test
session and ii) ``mysql`` creates a database that is thrown away at the end of
the each test function. The test functions looks like this:

```python
def test_get_obsnumber(mysql_proc, mysql):
    '''try to get the observation number'''
    # connect to the database
    # add my_table and one entry with obsnum = 42
    # create a configuration object
    conf = ...

    obs_num = get_obsnumber(conf)
    assert obs_num == 43
```

With a test function ready, I was ready to run ``pytest``. Of course I had to
install a couple of MySQL administration packages to be able to start the
database server. Then I hit a wall: the plugin [doesn't support MySQL
v5.7](https://github.com/ClearcodeHQ/pytest-mysql/pull/) <sup
id="a1">[1](#f1)</sup> and my desktop computer has exactly that version. On an
other computer I have [MariaDB]() v10.2.10, a drop-in replacement for MySQL. I
soon abandoned hope: MariaDB and MySQL administration interfaces are likely
subtly different and ``pytest-mysql`` cannot properly set user names when
creating a new database.

During my searches I stumbled across
[my_virtualenv](https://github.com/evgeni/my_virtualenv), a nice bash script
that sets up a sort of virtual environment with a temporary MySQL server. It
did work very well on my system with MariaDB, but failed on MySQL because I have
v5.7.

## The docker way

So I needed to find a way to run a MySQL server that would work at least of my
systems, independently of the MySQL/MariaDB version, or even without them. Then
I realised that I already had the answer: [Docker](https://docker.com).

I found the
[``pytest-docker``](https://github.com/AndreLouisCaron/pytest-docker) plugin and
some examples on how to use [mysql docker
containers](https://severalnines.com/blog/mysql-docker-containers-understanding-basics)
and [docker
compose](https://github.com/bossbossk20/docker-compose-mysql/blob/master/docker-compose.yml).

So I sat down and started to explore how to create docker containers, connect to
the MySQL servers running in them and create tables and add data. Once I felt
confident, I wrote the test fixtures and functions that I needed. They looked
roughly like the following code:

```python
import yaml

@pytest.fixture(scope='session')
def docker_compose_file(tmpdir_factory):
    '''Temporary docker compose file'''
    compose_file = tmpdir_factory.mktemp('docker_files').join('docker-compose.yml')

    environment_dict = dict(MYSQL_ALLOW_EMPTY_PASSWORD='no',
                            MYSQL_ROOT_PASSWORD='test',
                            MYSQL_DATABASE='test_db',
                            MYSQL_USER
                            MYSQL_USER='mysql_user',
                            MYSQL_PASSWORD='mysql_password')
    compose_conf = {'services': {'mysql': {'image': 'mysql',
                                           'container_name': 'test_suite',
                                           'environment': environment_dict}
                                  }
                     }

    with compose_file.open('w') as f:
        yaml.dump(compose_conf, stream=f)

    return compose_file.strpath


@pytest.fixture
def mysql_table(docker_ip, docker_services):
    '''Create the database table, fill it and drop it after the test is done'''
    # connect to the database
    # add my_table and one entry with obsnum = 42

    yield

    # drop my_table


def test_get_obsnumber(docker_ip, mysql_table):
    '''try to get the observation number'''
    # create a configuration object
    conf = ...  # this also contain the docker_ip

    obs_num = get_obsnumber(conf)
    assert obs_num == 43
```

Now that I had a test, I could start implementing the new function.

## One more step

The requirement had been satisfied, but I still had a problem:

> How to test my code on my machines?

Of course the answer is: 

> start the MySQL docker container and fill the database

I could do it my hand every time that I need it. But I don't like this kind of
things: it's tedious and very error prone.

So I decided to invest some more time and, with the help of
[docker-py](https://docker-py.readthedocs.io/), I added a set of commands to
integrate the setup and teardown of a MySQL docker container and the creation of
``my_table`` with the rest of the project. I also dropped ``pytest-docker`` and
rewrote the docker and MySQL fixtures using those new features: this way I have
more flexibility and control over my tests and less code repetitions.

So now, when I want to execute the code on my computer I run

    my_project docker_mysql up

to start the docker image and

    my_project run

to execute the main part. When I'm done I can get rid of the docker container
with

    my_project docker_mysql down

## Footnotes

<b id="f1">1</b>&bull; ``mysql_install_db`` [has been
deprecated](https://dev.mysql.com/doc/refman/5.7/en/mysql-install-db.html) in
favour of ``mysqld --initialize/--initialize-insecure`` [↩](#a1)
