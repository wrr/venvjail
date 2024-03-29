# venvjail: Python virtual environment that jails executed programs.

**Development status: Planning**

## Overview

`venvjail` allows to install and run python programs without giving
them full access to the user's account. The goal is to limit harm a
compromised dependency can make without sacrificing the user’s
convenience. `venvjail` offers very similar workflow to the standard
Python `venv`, in most cases you should not notice and be affected by
the additional limitations applied in the jailed venv.

`venvjail` uses Linux namespaces and runs without root privileges. It
requires a Linux kernel with user namespaces enabled (for example,
Ubuntu 22 default kernel).

The following restrictions are applied to the jailed processes by default:

* The processes can write only to files in the directory subtree in
   which the `venvjail` was created. The exception are the
  `venvjail` activation and configuration files, which are not
  accessible in the jail.

* The whole content of the home directory is hidden in the jail with
  the exception of an explicit list of configuration files which are
  available read-only. By default this list includes `.bashrc`,
  `.profile`, `.bash_profile` and can be modified via a
  `~/.dirjailconfig` file.

* The processes are allowed to create new files in the home directory
  (for example program config files), but all these created files are
  in fact stored in the `./home` sub directory of the virtual
  environment.

* The processes can read and write to the `/tmp` directory. Outside of
  the jail this directory points to the `/tmp/dirjail-XXXXXX`.

* The process can only see processes that are executed in the same jail.

* Most files in `/proc` and all files in `/run/user/UID` are not
  readable in `venvjail`.

* Suid programs cannot be executed in the jail.

* All other files that you can normally read and execute are also
  readable and executable in the jail (but not writable). This can
  also be adjusted via the `~/.dirjailconfig` file.

Compared to running a program in a separate virtual machine or a
Docker instance, the following conveniences improve the user
experience:

* The jail runs on top of your current distribution, so you can run
  all the programs that you have available on your machine. Of course,
  these executed programs are also jailed, so for example `ls ~` will
  only list files visible in the jail.

* Your username is preserved in the jail. You are not a `root`, and
  files owned and created by you are listed as such in the jail.

* An explicitly selected list of config file are exposed to the
  jail in read-only mode. Shell configuration, helper functions,
  aliases continue to work normally while in the jail.

* Your current working directory path and most of the directory
  structure is preserved in the jail.

* File system modifications done in the jail are limited to the jailed
  directory subtree, so you can be sure that removing this program
  directory removes all the files that the program created. This helps
  to keep the system tidy.

# Installation and usage

To install, run:

    pip install venvjail

To create a virtual environment in a `venv` directory:

    python -m venvjail venv

To start a new shell with the jailed virtual environment activated:

    ./venv/bin/activate-new-shell

Or:

    source ./venv/bin/activate-new-shell


Like with the standard Python `venv`, you can run any program from the
`venv/bin` directly without activating `venv` first, and all these
programs are executed in the jail. For example:

      ./venv/bin/python
       >>> import os, pathlib
       >>> os.listdir(pathlib.Path.home());
       # You can see that most of the files in your home dir are hidden in the jail:
       ['.bashrc', '.profile, ...]


# Where is the `./venv/bin/activate` script and how to run scripts that source this file?

`/venv/bin/activate` script of the standard Python `venv` module
activates the virtual environment in the current shell, this is why
this script should be sourced and not executed. Because it is not
possible to jail an already existing process, `venvjail` needs to
create a new shell process. To avoid confusion the activation script
is called `./venv/bin/activate-new-shell`.

Once the `venvjail` is active, `venv/bin/activate` script becomes
available, sourcing it is a no-op, because the `venv` is already
activate, but it allows to run existing shell scripts that call
`source/venv/activate`.
