---
layout: post
title:  "Testing a MySQL function"
categories: python mysql docker tests TDD
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

## Create a temporary MySQL server.

At first I decided to give a go at
[pytest-mysql](https://github.com/ClearcodeHQ/pytest-mysql): it is
[pytest](https://docs.pytest.org) plugin and it provides a couple of fixtures:
``mysql_proc`` runs a server for the duration of the test session and ``mysql``
creates a database that is thrown away at the end of the test function. The test
functions looks like this:

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

Having a test function I tried to run ``pytest``. It took me a few trials to
figure out that I needed to install a couple of packages () and then I hit a
wall: the plugin [doesn't support MySQL
v5.7](https://github.com/ClearcodeHQ/pytest-mysql/pull/) <sup
id="a1">[1](#f1)</sup>. On an other computer I have MariaDB v10.2.10 instead:
although this is a drop-in replacement for MySQL, it seems that the admin
interface is different enough not to work with ``pytest-mysql``.

I also found [my_virtualenv](https://github.com/evgeni/my_virtualenv), a nice
bash script that sets up up a sort of virtual environment with a temporary MySQL
server: it did work very well on my system with MariaDB, but failed on MySQL for
the reason described above.

## The docker way

So I needed to find a way to run a MySQL server that would work on every system,
independently of the MySQL/MariaDB version, or even without them: this seemed
the perfect job for [Docker](https://docker.com).

Before starting, I searched for more information and found the
[``pytest-docker``](https://github.com/AndreLouisCaron/pytest-docker) plugin and
an example on how to use [docker compose to setup a MySQL
container](https://github.com/bossbossk20/docker-compose-mysql/blob/master/docker-compose.yml).
After some exploratory work, I managed to get a first version of the test fixtures
and functions. Here is a pseudo-code example:

```python
import yaml

@pytest.fixture(scope='session')
def docker_compose_file(tmpdir_factory):
    '''Temporary docker compose file'''
    compose_file = tmpdir_factory.mktemp('arc_shot').join('docker-compose.yml')

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

    # drop the my_table


def test_get_obsnumber(docker_ip, mysql_table):
    '''try to get the observation number'''
    # create a configuration object
    conf = ...  # this also contain the docker_ip

    obs_num = get_obsnumber(conf)
    assert obs_num == 43
```

for more information and found a [nice
tutorial](https://severalnines.com/blog/mysql-docker-containers-understanding-basics)


## Footnotes

<b id="f1">1</b>&bull; ``mysql_install_db`` [has been
deprecated](https://dev.mysql.com/doc/refman/5.7/en/mysql-install-db.html) in
favour of ``mysqld --initialize/--initialize-insecure`` [↩](#a1)
