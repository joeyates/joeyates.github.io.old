---
layout: post
title: "My Everyday find Command"
date: 2012-12-23 14:08
comments: true
description: a shell funtion to quickly search for files
categories: shell
---
When I'm searching for files, I use this function:

```bash
# f - everyday find
# usage:
#   f filename_fragment [path]
# skips whatever you list in _exclude_matches
_exclude_matches=(bundle .git *.pyc)
_excludes=''
for _match in $_exclude_matches; do
  _excludes="${_excludes}-name '$_match' -prune -o "
done

eval "
function my_everyday_find() {
  find \$2 \
    $_excludes \
    -name \"*\$1*\"    \
    -print;
}
"
unset _exclude_matches _excludes _match
alias f=my_everyday_find
```

It searches for files and directories with partial matches of the first parameter:
```bash
$ f 26
./db/migrate/20121003094826_add_foo_to_bar.rb
```

If I supply a second parameter, it is taken as the directory to search in:
```bash
$ f 26 ..
../api/db/migrate/20121003094826_add_foo_to_bar.rb
../redmine/db/migrate/20091017212644_add_missing_indexes_to_messages.rb
```

