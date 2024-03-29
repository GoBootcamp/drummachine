# Copyright 2014 The Go Authors.  All rights reserved.
# Use of this source code is governed by a BSD-style
# license that can be found in the LICENSE file.

FROM golang:1.4.1

# enable faketime
RUN apt-get update && apt-get install -yq --no-install-recommends patch
ADD enable-fake-time.patch /usr/src/go/
RUN patch /usr/src/go/src/runtime/rt0_nacl_amd64p32.s /usr/src/go/enable-fake-time.patch
ADD write-to-stderr.patch /usr/src/go/
RUN patch /usr/src/go/src/runtime/sys_nacl_amd64p32.s /usr/src/go/write-to-stderr.patch

# add fake file system
ADD fake_fs.lst /usr/src/go/
RUN cd /usr/src/go && go run misc/nacl/mkzip.go -p syscall fake_fs.lst src/syscall/fstest_nacl.go

# build go nacl tool chain
RUN cd /usr/src/go/src && GOOS=nacl GOARCH=amd64p32 ./make.bash --no-clean
RUN cd /usr/local/bin && curl -s -O https://storage.googleapis.com/gobuilder/sel_ldr_x86_64 && chmod 0755 sel_ldr_x86_64

# add and compile tour packages
RUN GOOS=nacl GOARCH=amd64p32 go get \
	code.google.com/p/go-tour/pic \
	code.google.com/p/go-tour/reader \
	code.google.com/p/go-tour/tree \
	code.google.com/p/go-tour/wc

# add and compile sandbox daemon
ADD . /go/src/sandbox/
RUN go install sandbox

EXPOSE 8080
ENTRYPOINT ["/go/bin/sandbox"]
