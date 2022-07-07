# apt-transport-rclone

_An [rclone](https://rclone.org) transport method for the `apt` package management system_

[![Build Status](https://travis-ci.org/google/apt-golang-rclone.svg?branch=master)](https://travis-ci.org/google/apt-golang-s3)
[![Go Report Card](https://goreportcard.com/badge/github.com/google/apt-golang-s3)](https://goreportcard.com/report/github.com/google/apt-golang-s3)
[![GoDoc](https://godoc.org/github.com/google/apt-golang-s3?status.svg)](https://godoc.org/github.com/google/apt-golang-s3)

The apt-transport-rclone project is a fork of [apt-golang-s3](https://github.com/google/apt-golang-s3) and provides support for hosting private
[apt](https://en.wikipedia.org/wiki/APT_(Debian)) repositories in
[rclone-compatible](https://rclone.org) storage interfaces. This is useful if you have private
packages, vendored public packages, or forks of public packages that your
software or business depend on. This project started as a fork of an s3-specific package, but has been rewritten to use rclone instead.

## TL;DR
1. Build the binary `$ go build -o apt-transport-rclone main.go`
1. Install the binary `$ sudo cp apt-transport-rclone /usr/lib/apt/methods/rclone`
1. Add your rclone-based source to a package list `$ echo "deb rclone://storage-endpoint/private-repo-bucket stable main" > /etc/apt/sources.list.d/private-repo.list`
1. Update and install packages `$ sudo apt-get update && sudo apt-get install your-private-package`

## Building a debian package

For convenience, there is a `goreleaser` config for building debian archives.

```
goreleaser build
```

## Installing in production

The `apt-transport-rclone` binary is an executable. To install it copy it to
`/usr/lib/apt/methods/rclone` on your computer.


## Configuration
### APT Repository Source Configuration

```
$ cat /etc/apt/sources.list.d/my-private-repo.list
deb [trusted=yes] rclone://<storage-instance-name>/<bucket-or-folder> stable main
```

### APT Method Configuration

Configuration of this apt method should be done by creating a standard
rclone config file, `/root/.config/rclone/rclone.conf`.

## How it works

Apt creates a child process using the `/usr/lib/apt/methods/s3` binary and
writes to that processes standard input using a specific protocol. The method
interprets the input, downloads the requested files, and communicates back to
apt by writing to its standard output. The protocol spec is available here
[http://www.fifi.org/doc/libapt-pkg-doc/method.html/ch2.html](http://www.fifi.org/doc/libapt-pkg-doc/method.html/ch2.html).

## Similar Projects
* [https://github.com/google/apt-golang-s3](https://github.com/google/apt-golang-s3)
* [https://github.com/kyleshank/apt-transport-s3](https://github.com/kyleshank/apt-transport-s3)
* [https://github.com/brianm/apt-s3](https://github.com/brianm/apt-s3)
* [https://github.com/BashtonLtd/apt-transport-s3](https://github.com/BashtonLtd/apt-transport-s3)
* [https://github.com/lucidsoftware/apt-boto-s3/](https://github.com/lucidsoftware/apt-boto-s3/)

