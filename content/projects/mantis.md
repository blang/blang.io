---
title: Mantis - ArmA3 Server management
date: "2014-08-11"
lang: "go, Angular.js"
license: "MIT"
group: secondary
---

Mantis is one of the largest project realised till today. Mantis is used to manage ArmA2/3 gameservers. It manages the whole workflow from running, configuring multiple server instances to clients joining with auto configured mods. It's in production since April 2014.

<!--more-->

 It consists of multiple components:

- Mantis (Closed Source): Server management and configuration backend written in golang
- [Mantis Webfrontend](https://github.com/blang/mantis-webfrontend): Webfrontend of mantis (AngularJS)
- [Hornet](https://github.com/blang/hornet): Client desktop application used to join server instances with remote-configured settings like mods (using Mantis)
- [Hornet Webfrontend](https://github.com/blang/hornet-webfrontend): Webfrontend of hornet (AngularJS)
- [Stinger](https://github.com/blang/stinger): Proxy/Overlay backend of mantis to support multiple mantis instances on different hardware combined
- [pushr](https://github.com/blang/pushr): Release management and auto-update server for `hornet`