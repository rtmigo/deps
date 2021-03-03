# [depz](https://github.com/rtmigo/depz)

[![Generic badge](https://img.shields.io/badge/ready_for_use-no-red.svg)](#)
[![Actions Status](https://github.com/rtmigo/depz/workflows/CI/badge.svg?branch=master)](https://github.com/rtmigo/depz/actions)
[![Generic badge](https://img.shields.io/badge/CI_OS-MacOS,_Ubuntu-blue.svg)](#)
[![Generic badge](https://img.shields.io/badge/CI_Python-3.8,_3.9-blue.svg)](#)


Command line tool for symlinking dirs with reusable code into the working project.

# Motivation

**Reusing code** should be simple. If I have the needed code in a **nearby directory**, 
I just want to **include it** into the project. Without packaging it as a library 
for distribution or messing with IDE settings.

So I will probably just **create a symlink**:

```bash
$ ln -s /abc/libs/mylib /abc/myproject/mylib
```

Now `myproject` sees `mylib` as a local directory `myproject/mylib`. I can edit both `myproject` 
and `mylib` while working on `myproject`.

`depz` automates the creation of such symbolic links, solving emerging problems:

- **Portability**. To make the symlinks easy to recreate on another system
- **Dependencies**. To include not only `mylib`, but all the dependencies of `mylib`

# Usage

- Specify dependencies in `depz.txt`
- Run the command `depz`

# Specify dependencies

File `xxx/depz.txt` lists dependencies for `xxx`:
- `/abc/myproject/depz.txt` for `myproject`
- `/abc/libs/mylib/depz.txt` for `mylib`

The `depz.txt` format:
```sh

# lines prefixed by hashes are ignored

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
### Local dependencies are recursive

When a project depends on local `mylib`, it means, it also depends on all 
the dependencies of `mylib`. So after scannig `myproject/depz.txt` we will also 
scan `mylib/depz.txt` the same way.

### Paths are relative to parent of depz.txt

But resulting 
links will always reside in the project dir. For example, when running in `/abc/proj`:

| File  | Line | Resolves to | Creates symlink |
|--------------------|------------|---------------|--------|
|/abc/myproject/depz.txt|/abc/libs/xxx|/abc/libs/xxx|/abc/myproject/xxx|
|/abc/myproject/depz.txt|../libs/xxx|/abc/libs/xxx|/abc/myproject/xxx|
|/abc/libs/xxx/depz.txt|../zzz|/abc/libs/zzz|/abc/myproject/zzz|
 
# Run

```bash
$ cd /abc/myproject
$ depz link
```