--- 
layout: post
typo_id: 43
title: Show your Git Branch Name in your GNU Screen Status Line
---
# Git Branch Prompt

[Many](http://github.com/guides/put-your-git-branch-name-in-your-shell-prompt) [people](http://github.com/lvv/git-prompt/) [have](http://www.jukie.net/bart/blog/20080404105620) written about showing the current git branch in your shell prompt.

There is even the Bash function [__git_ps1](http://repo.or.cz/w/git.git?a=blob_plain;f=contrib/completion/git-completion.bash) provided in the main Git repo.

I prefer to show info like the current time and current git branch in a GNU screen status line.

<!--more-->

# Screen Status

As I use screen constantly, I decided to take a different route: put the current branch name in my screen status line.

# Changing the Screen Current Directory

When you start screen, it remembers the current directory. Issuing 'cd' commands inside a shell does not change the value. The only way to change it is to issue a `screen` `chdir` command.

In order to know the current branch in a particular screen window, I needed to make screen's current directory change to follow the shell it is presenting.

I added the following to my .bashrc:
{% codeblock set_screen_path function - .bashrc.sh %}
function set_screen_path() {
  screen -X chdir "`pwd`"
}

case $TERM in
screen*)
  PROMPT_COMMAND=set_screen_path
  ;;
esac
{% endcodeblock %}

This is Bash specific, but similar code should work in other shells.

# git-current-branch

Next, I created the following shell script:
{% codeblock Return the Current Git Path - git-current-branch.sh %}
#!/bin/sh

git branch 2>/dev/null | grep '*' | sed 's/\* //'
{% endcodeblock %}

# .screenrc

And finally, I added a 'backtick' command to my .screenrc file:

    startup_message off

    backtick 1 0 1 git-current-branch

    hardstatus alwayslastline "%-w%{.bw}%n %t%{-}%+w %-45= %1`"

    screen -t bash 0
    screen -t edit 1
    select 0

The important line is:
    backtick 1 0 1 git-current-branch

which creates a '[backtick command](http://www.gnu.org/software/screen/manual/html_node/Backtick.html)' (number 1) which calls `git-current-branch` once a second via <code>%1\`</code> in the hardstatus line.

#	The Result

{% img /images/GnuScreenShowingCurrentGitBranch.png 341 153 %}

# Conclusion

This system is not perfect as the shell only notifies screen of its directory when a new prompt is created.

The result is that if you switch screens, the status line doesn't get updated until after you issue a command.
