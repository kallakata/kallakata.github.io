---
title: "Kubernetes CLI parser"
excerpt: "A simple parser for Kubernetes metadata in a pretty table output."
layout: page
collection: portfolio
---

[Kubernetes CLI](https://github.com/kallakata/k8s_cli)

---

# A Kubernetes CLI parser with pretty output

:wave: A full fledged **Kubernetes CLI**.

As of now, it can parse _Pods, Namespaces, Clusters, Nodepools_ (in table output), and partially most important _metadata_ about clusters and pods. Output can be customized, default is a simple listing, otherwise it is in a form of table + interactive prompt.

Please refer to the official Google and/or Azure docs for more extensive metadata parsing and response properties related to clusters/nodepools. Authentication is handled locally via _kubeconfig_ and _gcloud_. Partial support for **Azure** has been added, temporarily only for clusters, authentication is performed via CLI.

**More features TBD.**

## TODO

- _Deployment_

## Usage

### Manual

```console
$ go build . && go install
$ ./app
$ ./app [COMMANDS] [FLAGS]
$ ./app --help
```

### Install binary

```console
$ go install .
```

### Build

```console
$ make build
$ cd ./bin
$ ./kubeparser [ARGUMENTS] [FLAGS]
```

### Run

```console
$ make run [COMMANDS] [FLAGS]
$ make run --help
```

### If you want to use new packages

```console
$ make vendor
```

### Test & format

```console
$ make fmt
$ make test
```
