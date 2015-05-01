---
layout: post
title:  "Ruby development with rbenv"
date:   2015-05-01
tags: [ruby,rbenv]
---

`rbenv` is a tool that manage multiple Ruby version environment. It's a simple, lightweight alternative to `RVM`.

###How it works
`rbenv` intercept Ruby commands using `shim`, which located in the `~/.rbenv/shims`, executables injected to the OS `PATH`, determines which Ruby version is used and pass the commands to that Ruby version installation. Each version of Ruby is installed in the `~/.rbenv/versions` directory.

###Understanding PATH
When some command like `ruby` or `rails` is invoked, the operating system search through a list of directories to find an executable file with that name. This list of directories lives is maintained in an environment variable called `PATH`, with each directory in the list is separated by a colon:

    /usr/local/bin:/usr/bin:/bin

Directories in `PATH` are searched from left to right, so a matching executable file in a directory at the beginning of the list takes precedence over another one at the end. The `/usr/local/bin` directory will be searched first, then `/usr/bin`, then `/bin` in the previous PATH variable.

###Understanding Shims
`rbenv` works by inserting a directory of shims in front of the `PATH` variable:

    ~/.rbenv/shims:/usr/local/bin:/usr/bin:/bin

Through the _rehashing_ process, rbenv maintains shims in the `~/.rbenv/shims` directory to match every Ruby command across every installed version of Ruby like: `irb`, `gem`, `rake`, `bundle`, `rails`, `ruby` ...

Shims are lightweight executables that simply pass these command to rbenv. So with rbenv installed, when a command like `rake` is called, the operating system will do the following:

  - Search through the `PATH` variable for an executable file named `rake`

  - Find the rbenv shim named `rake` at the beginning of the `PATH`

  - Run the shim named `rake`, which in turn passes the command along to rbenv

All the shims have the same content:

```
#!/usr/bin/env bash
set -e
[ -n "$RBENV_DEBUG" ] && set -x

program="${0##*/}"
if [ "$program" = "ruby" ]; then
  for arg; do
    case "$arg" in
    -e* | -- ) break ;;
    */* )
      if [ -f "$arg" ]; then
        export RBENV_DIR="${arg%/*}"
        break
      fi
      ;;
    esac
  done
fi

export RBENV_ROOT="~/.rbenv"
exec "~/.rbenv/libexec/rbenv" exec "$program" "$@"
```

###Installation

**Note:** `rbenv` isn't compatible with RVM. If RVM got installed in the system, it need to be removed all references to it:

    rvm implode

This will remove the `rvm/` directory and all the rubies built within it.

    gem uninstall rvm

This will remove the final trace of rvm.

Check the `.bashrc`, `.profile` and `.bash_profile` files to make sure there aren't any trace of RVM as well.

Let's begin:

1. Cloning the GitHub repository into `~/.rbenv`:

        $ cd
        $ git clone git://github.com/sstephenson/rbenv.git .rbenv

2. Add ~/.rbenv/bin to the $PATH for access to the `rbenv` command-line utility.

        $ echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bash_profile

3. Add `rbenv init` to the shell to enable shims and autocompletion

        $ echo 'eval "$(rbenv init -)"' >> ~/.bash_profile

4. Restart the shell to have rbenv and autocompletion:

        $ exec $SHELL

**Ubuntu Desktop note**: Modify ~/.bashrc instead of ~/.bash_profile

###How rbenv hooks into the shell
`rbenv init` is the only command that load extra commands into the shell. It simply does:

1. Sets up the shims path. This can be manually done by prepending `~/.rbenv/shims` to the $PATH variable.

2. Installs autocompletion. Sourcing `~/.rbenv/completions/rbenv.bash` will set that up

3. Rehashes shims automatically (or run manually with `rbenv rehash`).

4. Installs the sh dispatcher. This allows `rbenv` and plugins to change variables in the current shell.

Run `rbenv init -` to see exactly what happens under the hood.

###Installing Ruby version
Optional: Install [ruby-build](https://github.com/sstephenson/ruby-build#readme), which provides the `rbenv install` command that simplifies the process of installing new Ruby versions. Then use it to install new Ruby version:

```
# list all available versions:
$ rbenv install -l

# install a Ruby version:
$ rbenv install 2.2.2
```

Or manually install Ruby:

1. Create new folder in home directory to keep Ruby version's source code in:

        $ cd
        $ mkdir rbversions

2. Download the latest version of the Ruby source code

        $ curl -O http://cache.ruby-lang.org/pub/ruby/2.2/ruby-2.2.2.tar.gz

3. Unpack the download then `cd` into the newly extracted folder:

        $ tar xzvf ruby-2.2.2.tar.gz
        $ cd ruby-2.2.2

4. Run configure, `make` and `make install` making sure the correct version number in the `./configure --prefix` path is entered:

        $ ./configure --prefix=$HOME/.rbenv/versions/2.2.2
        $ make
        $ make install

    Now, Ruby 2.2.2 is installed in the `~/.rbenv/versions/2.2.2` directory.

5. Run `rbenv rehash` to rebuild the shim binaries. It’s recommended to do this anytime a new Ruby binary is installed (new Ruby version or gem that comes with a binary).

###Uninstalling Ruby Versions
To remove old Ruby versions, simply `rm -rf` the directory of that Ruby version. To find the directory of a particular Ruby version, use the command:

    rbenv prefix `version(2.2.2)`

The [ruby-build](https://github.com/sstephenson/ruby-build#readme) plugin provides an `rbenv uninstall` command to automate the removal process.

###Ruby version control

When a shim is executed, `rbenv` determines which Ruby version to use by reading from the following sources, in this order:

1. The `RBENV_VERSION` environment variable, if specified. The `rbenv shell` command can be used to set this environment variable in the current shell session.

        rbenv shell 2.2.1

    Behind the scenes the `RBENV_VERSION` environment variable is set to point to the Ruby version which is specified in the command:

        $ echo $RBENV_VERSION
        2.2.1

2. The first `.ruby-version` file found by searching the current working directory and each of its parent directories until reaching the root of the filesystem. The `.ruby-version` in the current directory can be modified with the command `rbenv local`.

        $ cd ~/projects/foobar
        $ rbenv local 2.2.1

    This writes the version to a file .rbenv-version in the current directory:

        $ cat .rbenv-version
        2.2.1

3. The global `~/.rbenv/version` file is the last place for `rbenv` to look up. This file can be modified using the `rbenv global` command. If the global version file is not present, rbenv assumes the "system" Ruby (whatever version should be run) will be used.

        $ rbenv global 2.2.1

    This will write the Ruby version to `~/.rbenv/version`

        $ cat ~/.rbenv/version
        2.2.1

###Locating the Ruby Installation
Each Ruby version is installed into its own directory under `~/.rbenv/versions`. Example:

    ~/.rbenv/versions/2.2.1/
    ~/.rbenv/versions/2.1.0/
    ~/.rbenv/versions/1.9.3-p327/

To see all the versions of Ruby that have installed and available to rbenv there’s the `rbenv versions` command. This lists all the versions of Ruby that rbenv knows about and adds an asterisk next to the current active version based on the current context e.g. shell, local or global. Let's see how the output of this command changes when the context is changed as well.

```
$ cd
$ rbenv versions
  system
  2.1.2
* 2.2.1 (set by ~/.rbenv/version)

$ cd ruby_projects/foo
$ rbenv versions
  system
* 2.1.2 (set by ~/ruby_projects/foo/.ruby-version)
  2.2.1

$ rbenv shell 2.2.1
$ rbenv versions
  system
  2.1.2
* 2.2.1 (set by RBENV_VERSION environment variable)
```

Source: [Up and Running with rbenv](http://www.sitepoint.com/up-and-running-with-rbenv/), [rbenv home page](https://github.com/sstephenson/rbenv)
