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

When setting up a Golang working environment, broadly speaking, there are two ways: using a package manager or installing from source. Most of the time it's easier to use a package manager and by executing one or two commands, the entire working environment can be setup for you. Well, this time, I just would like to see how manual installation works.


## Overview

Let me first list all possible steps that I need for the installation:

1. Get the Golang source code from somewhere, maybe a Github repo
2. Build the Golang source into executable binaries
3. 😱 It seems that I need to have a compiler ready before Step 2
4. Set up a bunch of environment variables for Golang
5. Verify the installation


## Bootstrap (Step 1, 3, 4)

- Golang v1.4 was the last distribution in which the toolchain was written in C

```shell
# clone the go source repo into a folder named go1.4
git clone git@github.com:golang/go.git go1.4

# navigate to the source folder
cd go1.4

# checkout the release-branch.go1.4 branch
git checkout release-branch.go1.4
```

After unpacking the Golang v1.4 source code, `cd` into the `src` subdirectory, and set `CGO_ENABLED=0` in the envionment, and run `make.bash`

```shell
# navigate into the src directory
cd src

# set environment variable CGO_ENABLED to 0
CGO_ENABLED=0

# run make.bash
./make.bash
```

Once the Golang v1.4 has been unpacked, you need to set `GOROOT_BOOTSTRAP` to its directory, so that `$GOROOT_BOOTSTRAP/bin/go` is the `go` command to use for building the Golang source code. 
- ❗️ Do not attempt to reuse this git clone in the later steps
- ❗️ The go1.4 bootstrap toolchain must be able to properly traverse the go1.4 sources that it assumes are present under this repository root


## Fetch the source (Step 1 again 😓)

This time we need to get the copy of Golang source code that will be built. 

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

To build the Go distribution, just run the commands below, and once the commands finish executing, the installation is done. 

```shell
# navigate into the src subfolder
cd src

# run all.bash
./all.bash
```


## Test the installation

Check the current go version by executing the command below in a termianl:

```shell
go version
```

Check whether the installed Golang is able to build a "Hello, world!" project:
- Create a `main.go` file anywhere in your local computer
- Do something simple with Golang, e.g., printing "Hello, world!" to the terminal (shown below). 

```golang
package main

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

How do I change the current installed version to a different one or how do I experiment with my local changes to the Golang source code?

The answer is just rebuild the source. 

The code in the `$HOME/goroot/src` folder should reflect the version of Golang source code you want to build, otherwise `git checkout`. Then run

```shell
# navigate into the src subfolder
cd $HOME/goroot/src

# run all.bash
./all.bash
```