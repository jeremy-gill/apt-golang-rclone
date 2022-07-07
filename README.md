# apt-golang-rclone

_An [rclone](https://www.rclone.org) transport method for the `apt` package management system_

[![Build Status](https://travis-ci.org/google/apt-golang-rclone.svg?branch=master)](https://travis-ci.org/google/apt-golang-s3)
[![Go Report Card](https://goreportcard.com/badge/github.com/google/apt-golang-s3)](https://goreportcard.com/report/github.com/google/apt-golang-s3)
[![GoDoc](https://godoc.org/github.com/google/apt-golang-s3?status.svg)](https://godoc.org/github.com/google/apt-golang-s3)

The apt-golang-rclone project is a fork of [apt-golang-s3](https://github.com/google/apt-golang-s3) and provides support for hosting private
[apt](https://en.wikipedia.org/wiki/APT_(Debian)) repositories in
[rclone-compatible](https://www.rclone.org) storage interfaces. This is useful if you have private
packages, vendored public packages, or forks of public packages that your
software or business depend on. This project started as a fork of an s3-specific package, but has been rewritten to use rclone instead.

## TL;DR
1. Build the binary `$ go build -o apt-golang-rclone main.go`
1. Install the binary `$ sudo cp apt-golang-rclone /usr/lib/apt/methods/rclone`
1. Add your s3 based source to a package list `$ echo "deb rclone://storage-endpoint/private-repo-bucket stable main" > /etc/apt/sources.list.d/private-repo.list`
1. Update and install packages `$ sudo apt-get update && sudo apt-get install your-private-package`

## Building the go program

There is an included Dockerfile to setup an environment for building the binary
in a sandboxed environment:

```
$ ls
Dockerfile  main.go  method  README.md

$ docker build -t apt-golang-rclone .
...

$ docker run -it --rm -v $(pwd):/app apt-golang-rclone bash

root@83823fffd369:/app# ls
Dockerfile  README.md  build-deb.sh  go.mod  go.sum  main.go  method

root@83823fffd369:/app# go build -o apt-golang-rclone main.go
...

root@83823fffd369:/app# ls
Dockerfile  README.md  apt-golang-rclone  build-deb.sh  go.mod  go.sum  main.go  method

root@83823fffd369:/app# exit
exit

$ ls
apt-golang-rclone  build-deb.sh  Dockerfile  go.mod  go.sum  main.go  method  README.md
```

## Building a debian package

For convenience, there is a small bash script in the repository that can build
the binary and package it as a .deb.

```
$ ls
build-deb.sh  Dockerfile  go.mod  go.sum  main.go  method  README.md

$ docker build -t apt-golang-rclone .

$ docker run -it --rm -v $(pwd):/app apt-golang-rclone /app/build-deb.sh
...
Created package {:path=>"apt-golang-rclone_1_amd64.deb"}

$ ls
apt-golang-rclone  apt-golang-rclone_1_amd64.deb  build-deb.sh  Dockerfile  go.mod  go.sum  main.go  method  README.md
```

## Installing in production

The `apt-golang-rclone` binary is an executable. To install it copy it to
`/usr/lib/apt/methods/rclone` on your computer. The .deb file produced by
`build-deb.sh` will install the method in the correct place.


## Configuration
### APT Repository Source Configuration

We recommend issuing a separate set of API keys, with read-only access, to the
S3 bucket that hosts your repository. The keys are specified in the apt sources
list configuration as follows:

```
$ cat /etc/apt/sources.list.d/my-private-repo.list
deb s3://aws-access-key-id:aws-secret-access-key@s3.amazonaws.com/my-private-repo-bucket stable main
```

### APT Method Configuration

The current default AWS region is set to `us-east-1`, but can be overridden by
adding an option in your apt configuration, e.g.

```plain
echo "Acquire::s3::region us-east-1;" > /etc/apt/apt.conf.d/s3
```

Alternatively, you may specify an IAM role to assume before connecting to S3.
The role will be assumed using the default credential chain; this option is
mutually exclusive with static credentials in the S3 URL.

```plain
echo "Acquire::s3::role arn:aws:iam::123456789012:role/s3-apt-reader;" > /etc/apt/apt.conf.d/s3
```

Additional configuration options may be added in the future.

## How it works

Apt creates a child process using the `/usr/lib/apt/methods/s3` binary and
writes to that processes standard input using a specific protocol. The method
interprets the input, downloads the requested files, and communicates back to
apt by writing to its standard output. The protocol spec is available here
[http://www.fifi.org/doc/libapt-pkg-doc/method.html/ch2.html](http://www.fifi.org/doc/libapt-pkg-doc/method.html/ch2.html).

## Similar Projects
* [https://github.com/kyleshank/apt-transport-s3](https://github.com/kyleshank/apt-transport-s3)
* [https://github.com/brianm/apt-s3](https://github.com/brianm/apt-s3)
* [https://github.com/BashtonLtd/apt-transport-s3](https://github.com/BashtonLtd/apt-transport-s3)
* [https://github.com/lucidsoftware/apt-boto-s3/](https://github.com/lucidsoftware/apt-boto-s3/)

## Disclaimer
This is not an officially supported Google product.
