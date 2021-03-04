# [depz](https://github.com/rtmigo/depz)

[![Generic badge](https://img.shields.io/badge/ready_for_use-no-red.svg)](#)
[![PyPI version shields.io](https://img.shields.io/pypi/v/depz.svg)](https://pypi.python.org/pypi/depz/)
[![Actions Status](https://github.com/rtmigo/depz/workflows/CI/badge.svg?branch=master)](https://github.com/rtmigo/depz/actions)
[![Generic badge](https://img.shields.io/badge/CI_OS-MacOS,_Ubuntu-blue.svg)](#)
[![Generic badge](https://img.shields.io/badge/CI_Python-3.7--3.9-blue.svg)](#)


Command line tool for symlinking directories with reusable source code into the project.

Language-agnostic.

# Why

**Reusing code** should be simple. If I have the needed code in a **directory on a local drive**, 
I just want to **include it** into the project. Without packaging it as a library 
for distribution or messing with IDE settings.

My first thought is to **create a symlink**:

```bash
$ ln -s /abc/libs/mylib /abc/project/mylib
```

Now `project` sees `mylib` as a local directory `project/mylib`. I can edit both `project` 
and `mylib` while working on `project`.

But here problems arise:
- **Portability**. How to make the symlinks easy to recreate on another system?
- **Recursive local dependencies**. How to include not only `mylib`, but all the dependencies of `mylib`?

The answer is `depz`. It reduces these tasks to a one-line command.

# Install

Get a working [Python](https://www.python.org/) ≥3.7 and [pip](https://pip.pypa.io/en/stable/installing/). You may also need a computer. Then:

```bash
$ pip3 install depz
```

Make sure that it is installed:

```bash
$ depz --help
```

Upgrade it later:
```bash
$ pip3 install depz --upgrade
```



# Use

- Specify dependencies in `depz.txt`
- Run the command `depz`

# Specify dependencies

File `xxx/depz.txt` lists dependencies for `xxx`:
- `/abc/myproject/depz.txt` for `myproject`
- `/abc/libs/mylib/depz.txt` for `mylib`

The `depz.txt` format:
```sh
# lines that specify local directory 
# names are LOCAL dependencies:
  
/absolute/path/to/mylib1
../libs/mylib2
~/path/mylib3

# lines that cannot be resolved to an existing 
# directory are considered EXTERNAL dependencies
 
requests
numpy
```

# Run

```bash
$ cd /abc/myproject
$ depz
```

This recursively scans `/abc/myproject/depz.txt` and prints all the found dependencies. Doesn't make any changes to the file system. 

---------

```bash
$ cd /abc/myproject
$ depz --relink
```

Removes all the symlinks found in `/abc/myproject`. Adds new symlinks to the local dependencies. Prints external dependencies.
 

# Local dependencies

### They are recursive

When a project depends on local `mylib`, it means, it also depends on all 
the dependencies of `mylib`. So after scannig `myproject/depz.txt` we will also 
scan `mylib/depz.txt` to include its dependencies too.

### Paths are relative to the current depz.txt

When we scan `/abc/myproject/depz.txt`, the paths are relative to `/abc/myproject`. Then we found a link 
to `mylib` and started scanning `/abc/mylib/depz.txt`. The paths found there are relative to `/abc/mylib/depz.txt`.  

But all the symlinks will go directly into `/abc/myproject`.

The following examples show how the directories will be linked when running `depz` for `/abc/project`:

#### Default behavior

| File  | Line | Resolves to | Creates symlink |
|--------------------|------------|---------------|--------|
|/abc/project/depz.txt|/abc/libs/xxx|/abc/libs/xxx|/abc/project/xxx|
|/abc/project/depz.txt|../libs/xxx|/abc/libs/xxx|/abc/project/xxx|
|/abc/libs/xxx/depz.txt|../zzz|/abc/libs/zzz|/abc/project/zzz|

#### With `--mode=layout`

| File  | Line | Resolves to | Creates symlink |
|--------------------|------------|---------------|--------|
| /abc/project/depz.txt | /abc/libs/xxx|/abc/libs/xxx/src<br/>/abc/libs/xxx/test|/abc/project/src/xxx<br/>/abc/project/test/xxx |
| /abc/project/depz.txt | ../libs/xxx|/abc/libs/xxx/src<br/>/abc/libs/xxx/test|/abc/project/src/xxx<br/>/abc/project/test/xxx |
| /abc/libs/xxx/depz.txt | ../zzz|/abc/libs/zzz/src<br/>/abc/libs/zzz/test|/abc/project/src/zzz<br/>/abc/project/test/zzz |

This is useful for frameworks with strict directory structure such as Flutter.

# External dependencies

By default, the list of all external dependencies is simply printed to the terminal like that:

```txt
$ depz

Scanning /abc/myproject/depz.txt
...
External dependencies: pandas numpy requests
```

The `-e` argument causes the command to print only the list of dependencies.

## Print in one line:

```txt
$ depz -e line

pandas numpy requests
```

This can be useful for installing Python external depedencies:
```txt
$ pip3 install $(depz -e line)
```

Or install external dependencies and symlink local ones:
```txt
$ pip3 install $(depz -e line --relink)
```


## Print one per line:
```txt
$ depz -e multi

pandas
numpy
requests
```

Sample usage for creating pip-compatible file:

```txt
$ depz -e multi > requirements.txt
```