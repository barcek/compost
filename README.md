# compost

Recycle Docker commands.

A CLI tool for command storage, rerun and modification, with type annotation and a self-test.

## Why?

For the flexibility of single containers with more of the preset approach offered by Compose.

Useful for learning what Docker can do without the initial overhead of remembering specific commands, and, once comfortable, as a tool for iterating towards a preferred container set.

## How?

Prefacing a supported Docker command with `compost` (see [Getting started](#getting-started) below) stores the command, prints an ID to the terminal for later reference then runs the command as usual. For example, for `docker run`:

```shell
compost docker run <remaining_args>
```

This prints the storage ID, specifically '1' if the command run was the very first. This ID allows the Docker command to be rerun using `compost` and the Docker keyword only, i.e. avoiding the need to locate the previously run command in history or recreate the set of arguments.

For example, if the command was a `Docker run` command and the ID was in fact '1':

```shell
compost run 1
```

### Rerunning with changes

A stored command can be rerun with one or more of its arguments changed, by passing after the ID only the arguments to be updated.

```shell
compost run <ID> <args_for_update>
```

This retains the original command in storage, with the same ID, but stores the modified version under its own new ID, printing the new ID then running the modified command.

Positional arguments passed in this way are replaced, as are options taking their own arguments, while boolean options are toggled. For example, take this initial command:

```shell
compost docker run -de TEST=test test:1.0.0
```

If the ID printed were '1', the command could be rerun with the 'detach' option unset and the image string updated as follows:

```shell
compost run 1 -d test:1.0.1
```

This would store, print the ID for and run the following modified command:

```shell
compost docker run --env TEST=test test:1.0.1
```

For commands with more than one positional argument, when changing the second or later positional argument any preceding positional arguments should be passed too. As above, multipart positional arguments such as commands should be passed as a single string.

Options are available allowing the command to be printed and/or deferred (see [Options](#options) below).

### Storage of commands

The commands are stored in an SQLite database created by default in the '/home' directory for the user, per the `$HOME` environment variable, with the default filename '.compost.db'. For testing, a temporary database is created in the same directory with the default filename '.compost_test.db'.

The full paths and table creation strings are set close to the top of the source file.

### Supported arguments

Currently supported are the keywords `run` and `exec` with positional arguments and a subset of the options offering a short flag, plus `--name` and `--rm`. Any multipart positional argument, such as a command with its own arguments, should be passed quoted.

The flags available and action performed for each by `ArgumentParser` are defined close to the top of the source file.

### Listing and deletion

A command can be deleted from the store by including the argument `rm`:

```shell
compost <keyword> rm <ID>
```

All commands stored can be listed by including `ls`:

```shell
compost <keyword> ls
```

Usage can be seen with `compost help`, or with the `--help` or `-h` flag.

## Options

The following can be passed to `compost` either a) before a Docker command or b) after the `exec` or `run` keyword when rerunning with or without changes:

- `--defer` / `-d`, to store the command without running
- `--print` / `-p`, to show the command stored

## Getting started

The source file is written in Python 3.11 using the standard library only, with type annotation.

On a Linux system with a compatible version of Python installed, the source file can be run with the command `python3 compost` while in the same directory, and from elsewhere using the pattern `python3 path/to/compost`.

Alternatively, it can be run with only `./compost` / `path/to/compost`, by first making it executable, if not already, with `chmod +x compost`. Once executable, it can be run from any directory with the simpler `compost` by placing it in a directory listed on the `$PATH` environment variable, e.g. '/bin' or '/usr/bin'.

The hashbang at the top of the file assumes the presence of Python 3.11 in '/usr/bin', the source code that Docker itself is installed and can be invoked directly with `docker`.

## Code verification

The test cases can be run with `compost test` (see [Getting started](#getting-started) above). The type annotation can be checked with Mypy, e.g. using the command `mypy --python-version 3.11 compost`.

## Development plan

- extend rerun, listing and removal to accept multiple IDs
- allow for ID reuse
- implement command name provision and reference
- allow for independent positional argument and argument part update
- include outstanding options to the Docker commands currently supported
- support additional Docker commands
- extend the database schema for context and provide keywords for inspection and analysis
- generalize the classes to allow for use with other tools and extend to package format
- improve error handling
- extend testing
- add comments
