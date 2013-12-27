---
layout: post
title: "My Everyday find Command"
date: 2012-12-23 14:08
comments: true
categories: 
---
When I'm searching for files, I use this function:

<script src="https://gist.github.com/4363381.js"></script>

It searches for files and directories with partial matches of the first parameter:
```
$ f 26
./db/migrate/20121003094826_add_foo_to_bar.rb
```

If I supply a second parameter, it is taken as the directory to search in:
```
$ f 26 ..
../api/db/migrate/20121003094826_add_foo_to_bar.rb
../redmine/db/migrate/20091017212644_add_missing_indexes_to_messages.rb
```

