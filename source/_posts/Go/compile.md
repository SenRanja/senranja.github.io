---
title: compile & packaging
date: 2024-08-29 07:12:57
tags:
top: 10
categories:
- Go
---

This article contains tentatively golang's packaging techniques and UPX repackaging introduction, because both these 2 topics have small content size and I'd better merge them into one article for typesettings.

# Golang Compiling cross platforms

Normally these commands could compile golang program:

    SET CGO_ENABLED=0
    SET GOOS=linux
    SET GOARCH=amd64
    go build main.go

Linux on x86:

    CGO_ENABLED=0 
    GOOS=linux 
    GOARCH=amd64 
    go build

Linux on arm:

    CGO_ENABLED=0
    GOOS=linux
    GOARCH=arm
    go build filename.go

Windows:

    CGO_ENABLED=0
    GOOS=windows
    GOARCH=amd64
    go build filename.go

MacOS:

    CGO_ENABLED=0
    GOOS=darwin
    GOARCH=amd64
    go build filename.go

CGO is a option for unification of C,C++ and golang, it can unify C and golang's basic data struct. Its default value is 0, and if you want to set it up to 1, refer to https://stackoverflow.com/questions/64531437/why-is-cgo-enabled-1-default. 

Normally if the target executive Linux has standard `libc`, `glibc`, you donnot enable this option. If enabling CGO, and the file size after packaging would be bigger.

# Golang Compiling without symbols

Golang's compiling tool chain would build the binaries staticly with standard libraries and Third-party libraries. And the binaries contain runtime and GC(Garbage Collection) instructions. Normally it is hard to analyse the binaries for reverse engineering.

go build command:

    go build -ldflags "-s -w"
    go build -ldflags "-s -w" -trimpath

    go build -ldflags "-w -s"  -o SecretDetection-lackOfOptimize.exe
    go build -ldflags "-s -w" -trimpath  -o SecretDetection-lackOfOptimize.exe

# UPX compresses the binaries

I recommand to read the raw github and download the upstream binaries: https://github.com/upx/upx/releases . In china, there are many modified UPX prevalent in many security platforms with backdoors, so that be vigilant except the original github project.

UPX is simple to use, and I dont have deeper research.

    # Often command
    upx -9 fscan.exe

    # The 2 commands below are possible to package with failure, and I suggest to do more tests after compression
    upx --best program.exe
    upx --brute program.exe
