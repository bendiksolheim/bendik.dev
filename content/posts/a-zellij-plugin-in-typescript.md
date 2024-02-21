+++
title = "Writing a Zellij plugin in \"TypeScript\""
date = 2021-03-03
draft = true

+++

Ever since I found [https://zellij.dev](Zellij) a couple of years ago, I have been fascinated about the [https://zellij.dev/documentation/plugins](plugin system it uses). I am not sure if this plugin architecture is common or not, but it seems at least one other relatively modern application seems to use it: [https://lapce.dev](Lapce editor). The architecture is based on WASM, and uses a specific way of crafting WASM files called [https://wasi.dev](WASI): because WASM is mostly meant for a browser environment, there is no system interface standard. WASI is an attempt to standardize an interface for your WASM module so communication between the plugin and host is easier. Another cool thing is that plugins are not limited to being written in the same language as the host application. Zellij already supports plugins written in Rust and Go, but since I’m not at all fluent in any of them, let’s try "TypeScript" (more on the apostrophes shortly, I promise).

<!-- more -->

The plugin system was upgraded and stabilized somewhat recently, so I finally decided to write a plugin. This blog post is me documenting the journey.

As of the time of this blog post, there are two languages and frameworks to choose from: Rust (which is the officially supported one from the Zellij developers), and Go (which is implemented by someone else, I believe). I know none of these languages. Well actually, I once gave Rust a decent try, but my head didn’t really understand all of that ownership stuff any more than throwing around ampersands, asterisks, and writing `move` here and there. I have actually also had a look at Go once upon a time, but I really despise the error handling.

But as I wrote earlier, language doesn’t really matter: as long as there is a way to compile a language to a WASI compliant binary, you’re good. Unluckily, the process of compiling some languages to WASM is a bit complicated. It seems to me that some of the compilation processes are closer to the "this is theoretically possible" land than "this makes complete sense and is 100% documented". Throw in the concept of binding functions in WASM to functions exposed by the runtime, and you might say that I am quite far from being comfortable.

So I ended up picking ["TypeScript"](https://www.assemblyscript.org), which is a language I know well from my day job as a developer, and the whole toolchain is built to support WASM with an optional package for supporting WASI. In reality, AssemblyScript is not really TypeScript (which is the reason behind the apostrophes in the title), but it looks enough like it for me to feel comfortable in it. The differences are due to constraints in the WASM specification, which is hard to argue against.

## Getting started with WASM in AssemblyScript

Getting started is really easy: the [documentation](https://www.assemblyscript.org/getting-started.html#setting-up-a-new-project) is good, and because it uses `npm` as a package manager I can feel right at home.

```bash
$ npm init -y
$ npm install --save-dev assemblyscript
$ npx asinit .
```

This scaffolds a simple program in AssemblyScript which can add two numbers, all ready to compile and run! Let`s try it

```bash
# Let’s see our program first
$ cat assembly/index.ts
// The entry file of your WebAssembly module.

export function add(a: i32, b: i32): i32 {
  return a + b;
}

# Looks good – let’s build and run it
$ npm run asbuild
$ npm start
```

At this point, you can open your browser and see the magnificent number "3". Not really mind blowing stuff. But keep in mind that this is JavaScript loading a binary file, calling a function exposed from the binary file, and displaying the output in a browser. This is not really too different from compiling and running Doom with emscripten, except that that is about 999999 times more complicated I guess. But still, the concept is about the same.

## Leveling up from WASM to WASI

Running WASM in a browser does not really help us. We are not targeting a browser here, but Zellij, remember? To create a supported WASM file we need to adhere to the WASI standard, which AssemblyScript does not do by default. I am not at all an export on WASM and WASI, but as far as I understand it the WASI standard defines some useful things when you want to run a binary on a operating system instead of a browser: standard in and out, file access, exit codes, argv, and some other stuff.

To enable support for WASI, we need to do two things: install a library, and alter the build process slightly:

```bash
$ npm install --save-dev @assemblyscript/wasi-shim
```

Then, edit your `asconfig.json` file and add
```json
"extends": "./node_modules/@assemblyscript/wasi-shim/asconfig.json"
```

somehwere in the top level object. I have no idea what this actually does, other than making things work. Which is a good thing!

Let’s also change our program a bit. In the previous example, we loaded the WASM file with JavaScript and called the exposed function from JavaScript. This is possible in a WASI environment as well, but printing something is easier. Change your `assembly/index.ts` file to the following:

```typescript
console.log("Hello, world!")
```

You can now compile your project, and run it! But hold up a bit here. You can’t run it without a WASM runtime. AssemblyScript only compiles your code, it can’t run it. I installed [wasmtime](https://wasmtime.dev), and had a go at it

```bash
$ npm run asbuild
$ wasmtime build/release.wasm
Hello, world!
```

Nice. You just fired up a virtual machine and spent a shitload of CPU cycles to display "Hello, world!" in your terminal. Amazing.

## Enter Zellij

We now know how to compile a valid WASI file, so let’s take the next step and make something work inside Zellij. A WASI file is not enough by itself, we also need to adhere to the API expected by Zellij.

Understanding the API requires some detective work. The Rust and Go frameworks are fairly well documented, but the underlying API/"protocol" (I have no idea what the correct WASM term is here) is understandably not documented anywhere other than in the code. Which is completely fine, since 99.9% of those interested in developing a plugin will just use Rust or Go (or any other language supported in the future). And, to be honest, having to decode the inner workings of a Zellij plugin was fun! It took a few days to understand everything, but eventually I got there.

Let me spare you all the details about the debugging, and get straight to the details. Zellij expects your WASM file to expose at least three functions

```
// Called by Zellij to initialize your plugin
fn load(): void

// Called by Zellij every time "something" happens, so you can react to it
fn update(): bool

// Called by Zellij when `update` returns true, so you can render your new state
fn render(cols: u32, rows: u32): void
```

Implementing these should be enough to get Zellij to load your plugin, so let’s start with this! Again, edit your `assembly/index.ts` file with this content

```typescript
export function load(): void {
    console.log("load");
}

export function update(): bool {
    console.log("update");
    return false;
}

export function render(cols: u32, rows: u32): void {
    console.log("Hello from plugin!")
}
```

and compile

```bash
$ npm run asbuild
```

... and finally run it! But, hold on... This time we actually want to run it with Zellij, and not wasmtime, right? There is still one piece missing for this to work: a Zellij layout which loads the plugin. Let’s create a really simple one

```kdl
layout {
  pane {
    plugin location="file:build/debug.wasm" {
    }
  }
}
```

Finally! You can now launch Zellij with

```bash
$ zellij --layout ./layout.kdl
```

and be amazed! No, really, be! I am not joking this time. If we break down what really just happened, there are a lot of moving parts thrown in the mix to make this work. 

Body

