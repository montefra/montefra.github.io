---
layout: post
title:  "Testing a MySQL function"
categories: python mysql tests
author: Francesco Montesano
---

## How hard can it be?

### Systems 

* OpenSUSE Tumbleweed: mysql  Ver 15.1 Distrib 10.1.25-MariaDB, for Linux (x86_64) using readline 5.1
* (K)ubuntu 16.04:

### Software that helps

* [my_virtualenv](https://github.com/evgeni/my_virtualenv): bash script, works
  on OpenSUSE, fails on Kubuntu
* [pytest-mysql](https://github.com/ClearcodeHQ/pytest-mysql): pytest plugin. On
  OpenSUSE cannot start the server, because the use is set to ``None``, no
  matter what. On Kubuntu doesn't work because ``mysql_install_db`` [has been
  deprecated](https://dev.mysql.com/doc/refman/5.7/en/mysql-install-db.html) in
  favour of ``mysqld --initialize/--initialize-insecure`` and many packages has
  not yet been updated.

## The docker way

[example of docker mysql](https://severalnines.com/blog/mysql-docker-containers-understanding-basics)
[example of docker compose](https://github.com/bossbossk20/docker-compose-mysql/blob/master/docker-compose.yml)
