---
title: "Isalive"
date: 2019-02-22T08:16:52-05:00
draft: false

keywords:
- golang
- monitoring
- cli
---

# Building a HTTP monitoring cli appliclation with notifications via Gmail.

Due to the multi-role nature of startups, I tend to tackle a lot of the infrastructure and DevOps related tooling and management in addition to my more standard development work. To help monitor all of the various services and to experiment with Golang, I've created an [easy-to-use CLI application](https://github.com/christopherdiehl/isAlive) that can send an email if an endpoint returns a non 200 status code. Feel free to checkout the source code and let me know your thoughts!

Sample usage below:

```
usage: isalive [<flags>] <command> [<args> ...]

A command-line monitoring application.

Flags:
  --help  Show context-sensitive help (also try --help-long and --help-man).

Commands:
  help [<command>...]
    Show help.

  add <endpoint>
    Add a new endpoint to monitoring.

  remove <endpoint>
    Add a new endpoint to monitoring.

  scan [<alert>]
    Scans the stored hosts
```

