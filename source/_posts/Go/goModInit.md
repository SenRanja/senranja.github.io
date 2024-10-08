---
title: go mod init without cobra
date: 2024-08-24 14:26:04
tags:
categories:
- Go
---

I wrote an article about how **cobra** inits a golang mod, and I think maybe cobra is more common and frequent. This page is describing without cobra.

If you want to build a golang model:

1. create a directory, for example, I want to create a project named 'xxx', then just create it
2. cd into the directory, and execute command `go mod init github.com/senranja/xxx`, then you would see there is a `go.mod`
3. Then my case changes into another existent golang project.
4. how to `import modules` and update the `go mod`?

![GolangMod](GolangMod.png)

In goland, normally you can see in its builtin fire exploration, there is one only `go.mod`, and it actually represents both `go.mod` and `go.sum`. If you import exterior golang's mod, then you **right-click** the `go.mod` and `go mod tidy`, IDE automaticly create or update `go.sum`, dwelt with your code's new modules.

![gomodTidy](gomodTidy.png)

Normally, you dont need to focus what is `go mod` or `go module` during using goland, goland will help you to deal with the download of different exterior go module.

I will show you what are go.sum and go.mod. go.mod contains your name(in the init case I named the project xxx, and in the example files and pictures named gitlab-misconfig) and your module dependencies.

go.sum is a checkfile, it contains the dependencies' hash summary, to demostrate its raw dependencies' hash.

go.mod

```go.mod
module gitlab-misconfig

go 1.19

require (
	github.com/google/go-querystring v1.1.0
	github.com/hashicorp/go-cleanhttp v0.5.2
	github.com/hashicorp/go-retryablehttp v0.7.1
	github.com/sirupsen/logrus v1.9.0
	github.com/spf13/cobra v1.6.1
	github.com/spf13/viper v1.14.0
	github.com/stretchr/testify v1.8.1
	github.com/xanzy/go-gitlab v0.79.1
	github.com/xuri/excelize/v2 v2.7.0
	golang.org/x/oauth2 v0.3.0
	golang.org/x/time v0.3.0
)

require (
	github.com/davecgh/go-spew v1.1.1 // indirect
	github.com/fsnotify/fsnotify v1.6.0 // indirect
	github.com/pelletier/go-toml v1.9.5 // indirect
	github.com/pelletier/go-toml/v2 v2.0.5 // indirect
	github.com/pmezard/go-difflib v1.0.0 // indirect
	github.com/richardlehane/mscfb v1.0.4 // indirect
	github.com/richardlehane/msoleps v1.0.3 // indirect
	github.com/spf13/afero v1.9.2 // indirect
	github.com/spf13/cast v1.5.0 // indirect
	github.com/spf13/jwalterweatherman v1.1.0 // indirect
	github.com/spf13/pflag v1.0.5 // indirect
)
```

go.sum

```go.sum
cloud.google.com/go v0.26.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
cloud.google.com/go v0.34.0/go.mod h1:aQUYkXzVsufM+DwF1aE+0xfcU+56JwCaLick0ClmMTw=
cloud.google.com/go v0.38.0/go.mod h1:990N+gfupTy94rShfmMCWGDn0LpTmnzTp2qbd1dvSRU=
cloud.google.com/go v0.44.1/go.mod h1:iSa0KzasP4Uvy3f1mN/7PiObzGgflwredwwASm/v6AU=
cloud.google.com/go v0.44.2/go.mod h1:60680Gw3Yr4ikxnPRS/oxxkBccT6SA1yMk63TGekxKY=
cloud.google.com/go v0.44.3/go.mod h1:60680Gw3Yr4ikxnPRS/oxxkBccT6SA1yMk63TGekxKY=
```