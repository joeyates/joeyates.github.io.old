--- 
layout: post
typo_id: 35
title: Debian Alternatives
---
The Debian <a href="http://www.debian-administration.org/articles/91">Alternatives</a> system manages groups of symlinks, which can be modified en-masse.

For each program that it has to manage, Alternatives expects there to be a symlink from /usr/bin to /etc/alternatives/[PROGRAM NAME].

<!--more-->

For example, with vi, on my system:
    $ which vi
    /usr/bin/vi

    $ ls -ll /usr/bin/vi
    lrwxrwxrwx 1 root root 20 2009-11-03 15:33 /usr/bin/vi -&gt; /etc/alternatives/vi

    $ ls -l /etc/alternatives/vi
    lrwxrwxrwx 1 root root 16 2009-11-03 15:33 /etc/alternatives/vi -&gt; /usr/bin/vim.nox

    $ ls -l /usr/bin/vim.nox
    -rwxr-xr-x 1 root root 1825312 2009-09-21 13:23 /usr/bin/vim.nox

So, on my system, the command &#39;vi&#39; starts /usr/bin/vim.nox via 2 symlinks.

# Commands
## List Main Programs
    $ update-alternatives --get-selections

## Show setup for a certain program
    $ update-alternatives --display vi
    vi - auto mode
     link currently points to /usr/bin/vim.nox
    /usr/bin/vim.nox - priority 40
     slave vi.1.gz: /usr/share/man/man1/vim.1.gz
     slave vi.fr.1.gz: /usr/share/man/fr/man1/vim.1.gz
     slave vi.it.1.gz: /usr/share/man/it/man1/vim.1.gz
     slave vi.pl.1.gz: /usr/share/man/pl/man1/vim.1.gz
     slave vi.ru.1.gz: /usr/share/man/ru/man1/vim.1.gz
    /usr/bin/vim.tiny - priority 10
     slave vi.1.gz: /usr/share/man/man1/vim.1.gz
     slave vi.fr.1.gz: /usr/share/man/fr/man1/vim.1.gz
     slave vi.it.1.gz: /usr/share/man/it/man1/vim.1.gz
     slave vi.pl.1.gz: /usr/share/man/pl/man1/vim.1.gz
     slave vi.ru.1.gz: /usr/share/man/ru/man1/vim.1.gz
     Current `best&#39; version is /usr/bin/vim.nox.

## Set Up a Program
    $ sudo update-alternatives --install /usr/bin/[SYMLINK] [TITLE] /PATH/TO/PROGRAM 10
