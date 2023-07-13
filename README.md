# compost

Recycle Docker commands.

## Why?

For the flexibility of single containers with the more preset approach of Compose.

## How?

Prefacing a Docker run command with `compost` (see [Getting started](#getting-started) below) stores the Docker command in long form, prints an ID for later reference then runs the command as usual.

For example, for `docker run`:

```
compost docker run <remaining_args>
```

The ID printed allows the command to be rerun by keyword. For the example above:

```
compost run <ID>
```

The command can be deleted from the store with the additional argument `rm`, e.g. `compost run rm <ID>`, and all commands stored can be listed with `ls`, e.g. `compost run ls`. Usage can be seen with `compost help`, or with the `--help` or `-h` flag.

Currently supported are the `run` and `exec` tasks, for positional arguments and the subset of options which offer a short flag, plus `--name` and `--rm`. Multipart positional arguments such as commands should be passed as a single string. The flags available and action performed for each by `ArgumentParser` are defined close to the top of the source file.

### Rerunning with changes

A stored command can be rerun with changes by passing the arguments to be changed after the ID.

```shell
compost run <ID> ...
```

This retains the original command in storage, with the same ID, but stores the modified version under its own ID, printing that ID and running as usual.

Positional arguments passed in this way are updated, as are options taking their own arguments, while boolean options are toggled. For example, take this initial command:

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

### Storage of commands

The commands are stored in an SQLite database created by default in the '/home' directory for the user, per the `$HOME` environment variable, with the default filename '.compost.db'.

For testing, a temporary database is created in the same directory with the default filename '.compost_test.db'.

The full paths and table creation strings are set close to the top of the source file.

## Getting started

The source file is written in Python 3.11 using the standard library only, with type annotation.

On a Linux system with a compatible version of Python installed, the source file can be run with the command `python3 compost` while in the same directory, and from elsewhere using the pattern `python3 path/to/compost`.

Alternatively, it can be run with only `./compost` / `path/to/compost`, by first making it executable, if not already, with `chmod +x compost`. Once executable, it can be run from any directory with the simpler `compost` by placing it in a directory listed on the `$PATH` environment variable, e.g. '/bin' or '/usr/bin'.

## Code verification

The test cases can be run with `compost test` (see [Getting started](#getting-started) below). The type annotation can be checked with Mypy, e.g. using the command `mypy --python-version 3.11 compost`.

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
