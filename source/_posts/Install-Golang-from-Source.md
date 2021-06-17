---
title: Install Golang from Source
date: 2021-06-17 00:09:57
tags:
- Golang
- the-hard-way
categories: 
- blog
---

## Why?

When setting up a Golang working environment, broadly speaking, there are two ways: using a package manager/installer or installing from source. Most of the time it's much easier to use a package manager or installer and by executing one or two commands, the entire working environment is ready to use. Well, this time, I just would like to see how manual installation works.

## Overview

Let me first list all steps that I can think of for the installation:

1. Get the Golang source code from somewhere, maybe a Github repo
2. Build the Golang source into executable binaries
3. 😱 It seems that I need to have a Go compiler ready before Step 2 (**bootstrap**)
4. Install the executable binaries, there might be scripts doing this or there are standard places to put these binaries
5. A working Go installation depends on several environment varialbes, I may need to set up those variables
6. Verify the installation

Let me think through the process carefully.
### The result of installation

The result of a successful installation is a working Go environmant in which I can run different Go commands and download and build 3rd-party source code and working on my own code. This means that the OS of my computer can correclty locate the Go toolchain and the installed 3rd-party binaries, the archive files, and the project source files written in Golang. The Go environment uses environment vairables to help the OS to do this:
- `GOROOT`: the directory containing the Golang source code that implements the Golang programming language
- `GOBIN`: the directory to install binary files
- `GOPATH`: the directorying containing three subdirectories, i.e., `bin`, `pkg`, and `src`

When using the official Go installer ([link](https://golang.org/doc/install#download)) to do the installation, the Go distribution will be installed in `/usr/local/go` ([ref](https://golang.org/doc/install#install)) and its structure is something like this:

```shell
/
├─ usr/
│  ├─ local/
│  │  ├─ go/
│  │  │  ├─ bin/
│  │  │  ├─ pkg/
│  │  │  ├─ src/
```
This directory, `/usr/local/go`, is the "GOPATH" and it has three subdirectories containing the source files (`src`), archive files (`pkg`), and binary files (`bin`). 

The `/usr/local/go/bin` (or `$GOPATH/bin`) directory is also added to the `PATH` environment variable so that all go binaries are accessible from the terminal. 

### The compiler

The Go toolchain is written in Go or Golang is implemented in Golang. To build it, I need a Go compiler installed ([ref](https://golang.org/doc/install/source#go14)). This is the compiler that is used for "bootstrapping" the installation. This compiler can be a fully working Go toolchain already setup on my computer or a designated version of Go commands setup just for building and installing of Golang from source. 
- 😱 A working version of Golang means in order to build Go from source, I need to have a working version of Golang on my system. Well, this means, by default, I will not be using the build result as my Go toolchain
- 👻 A designated version of Go setup just for building and installing Golang requires the use of an environment variable named `GOROOT_BOOTSTRAP`, which directs the bootstrap process to the designated Go compiler. In this case, `$GOROOT_BOOTSTRAP/bin/go` should be the `go` command to use.

### The source

The source code of Golang is obtained by using Git, so I need to have Git installed locally. I need to find a good place to put the source code, and as suggested by the doc ([ref](https://golang.org/doc/install/source#fetch)), I can use `goroot` as the folder name. I need to run a build script to build this copy of source code and the Go tool chain will be installed in `<parent dir>/goroot/bin`.


## Bootstrap (Step 1 and 3)

I would like to have a version of Go toolchain just for bootstrapping the building process, so that I can always inspect the implementation of the installed Go toolchain.

```shell
# clone the go source repo into a folder named go1.4
git clone git@github.com:golang/go.git go1.4

# navigate to the source folder
cd go1.4

# checkout the release-branch.go1.4 branch
git checkout release-branch.go1.4
```

- Golang v1.4 was the last distribution in which the toolchain was written in C ([ref](https://golang.org/doc/install/source#go14))
    - This is still source code, but I don't need a Go compiler to built it (but I need a C compiler)

After unpacking the Golang v1.4 source code, `cd` into the `src` subdirectory, and set `CGO_ENABLED=0` in the envionment, and run `make.bash`

```shell
# navigate into the src directory
cd src

# set environment variable CGO_ENABLED to 0
CGO_ENABLED=0

# run make.bash
./make.bash
```

Once the Golang v1.4 has been unpacked, you need to set the `GOROOT_BOOTSTRAP` environment variable to its directory, so that `$GOROOT_BOOTSTRAP/bin/go` is the `go` command to use for building the Golang source code. 
- ❗️ Do not attempt to reuse this git clone in the later steps
- ❗️ The go1.4 bootstrap toolchain must be able to properly traverse the go1.4 sources that it assumes are present under this repository root


## Fetch the source (Step 1 again 😓)

This time I need to get the copy of Golang source code that will be built. 

```shell
# clone the source code into a folder, in this case, a folder named goroot
# with
git clone https://go.googlesource.com/go goroot
# or
git clone git@github.com:golang/go.git goroot

# navigate into goroot
cd goroot

# get all the tags (so that I can select a particular version)
git fetch --all --tags 

# checkout a particular tag, e.g., go1.16.3
git checkout go1.16.3
# or create a new branch from a tag
git checkout -b go1.16.3 go1.16.3
```

Go will be installed in the directory where it is checkout out. If Go is checked out in `$HOME/goroot`, executables will then be installed in `$HOME/goroot/bin`. 

The directory containing the Golang source code may have any name, but if that folder is `$HOME/go`, it will conflict with the default location of `$GOPATH`. 


## Build and install (Step 2)

To build the Go distribution, just run the commands below, and once the commands finish executing, the building part is done and the Go executables should be in `$HOME/goroot/bin`. 

```shell
# navigate into the src subfolder
cd src

# run all.bash
./all.bash
```

The "installation" is done by setting up environment variables ([ref](https://golang.org/doc/install/source#environment)):
- `GOROOT`: The root of the Go tree, often $HOME/go1.X. Its value is built into the tree when it is compiled, and defaults to the parent of the directory where all.bash was run. There is no need to set this unless you want to switch between multiple local copies of the repository.
    - The value should be the directory where the second copy of Go source code is check out into
- `GOPATH`: The directory where Go projects outside the Go distribution are typically checked out.
    - The value to this variable should be the parent directory of the `bin`, `pkg`, and `src` directory, where the `go install` result, library archives, and source dependencies are placed respectively.
- `GOBIN`: The directory where executables outside the Go distribution are installed using the go command. 
    - Most of the time it's just `$GOPATH/bin`

Because the Go toolchain (commands) should be available in terminals, `$GOROOT/bin` and `$GOBIN` should be added to the `PATH` environment variable.


## Test the installation

Once the environment variables are ready, I can test the installation. 

Check the current go version by executing the command below in a termianl:

```shell
go version
```

Check whether the installed Golang is able to build a "Hello, world!" project ([ref](https://golang.org/doc/code#Command)):
- Create a `main.go` file anywhere in your local computer
- Do something simple with Golang, e.g., printing "Hello, world!" to the terminal (shown below). 

```golang
package main

import "fmt"

func main() {
	fmt.Println("Hello, world!")
}
```

- In a terminal, navigate to the directory containing the `main.go` file and execute:

```shell
go run main.go
```

If you see "Hello, world!" gets printed in the terminal, the installation is good. 


## Update

How do I update the current installed version to a different one or how do I experiment with my local changes to the Golang source code?

The answer is: just rebuild the source. 

The code in the `$HOME/goroot/src` folder reflects the version of Golang source code I want to build, otherwise `git checkout` the correct version. Then run

```shell
# navigate into the src subfolder
cd $HOME/goroot/src

# run all.bash
./all.bash
```

## Tail

Now that I have a working Go environment, I can build a 3rd copy of Go source and play with it or I can have multiple Go versions installed at the same time ([ref](https://golang.org/doc/manage-install#installing-multiple)).