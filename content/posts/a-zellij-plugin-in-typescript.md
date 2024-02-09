+++
title = "Writing a Zellij plugin in \"TypeScript\""
date = 2021-03-03
draft = true

+++

Ever since I found [https://zellij.dev](Zellij) a couple of years ago, I have been fascinated about the [https://zellij.dev/documentation/plugins](plugin system it uses). I am not sure if this plugin architecture is common or not, but it seems at least one other relatively modern application seems to use it: [https://lapce.dev](Lapce editor). The architecture is based on WASM, and uses a specific way of crafting WASM files called [https://wasi.dev](WASI): because WASM is mostly meant for a browser environment, there is no system interface standard. WASI is an attempt to standardize an interface for your WASM module so communication between the plugin and host is easier. Another cool thing is that plugins are not limited to being written in the same language as the host application. Zellij already supports plugins written in Rust and Go, but since I’m not at all fluent in any of them, let’s try "TypeScript" (more on the ticks shortly, I promise).

<!-- more -->

Body

