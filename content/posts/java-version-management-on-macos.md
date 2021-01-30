+++
title = "Java Version Management on macOS"
date = 2021-01-30
+++

With the increased frequency of Java releases handling multiple versions on a single machine is more and more relevant. Solutions already exists, such as the widely used [jenv](https://www.jenv.be) – my goto tool for a long time. For a number of reasons, I recently decided to throw it out of my system and build a solution myself. The result is a few small functions which handles most of my needs.

<!-- more -->

_(Tldr; if you just want the code, [just scroll down](#a-solution))_

My biggest issue was the startup time on new shells. It’s a [known issue](https://github.com/jenv/jenv/issues/148), with several workarounds, but at least to me they all affect my startup time noticably. Now, I might be an impatient guy, I’ll admit that, but I just can’t stand spending time on something that I feel should happen instantly.

My second biggest issue was the troubleshooting. I practically live in my terminal, so problems have a big effect on my productivity. For that reason (and others, I’m sure), I tend to like simple solutions. Fewer blows and whistles, fewer weird problems. Without going into too much detail, I feel that jenv has made a few weird decisions that has ended up with me spending precious time debugging and troublehooting. I am sure these decisions are made for good reasons, so no hard feelings there, but it aligns badly with my love for simplicity.

## Requirements

So I implemented my own solution. Because as you know, the rational solution to spending too much time on a problem is spending even _more_. But hey, it resulted in me writing this blog post so I guess there’s that.

My needs are quite simple
- I want the `java` command always available
- I want the ability to switch, locally, between versions when needed
- Bonus: automatically switching version when entering a folder would be cool

.. and I believe that’s about it. New versions can easily be downloaded with Homebrew. Changing the version globally would be nice, but is not strictly neccesary.

Now, macOS comes with a little known commandline tool called `java_home`. It is located at `/usr/libexec/java_home`, which is not on your `PATH` by default, so I guess it might be hard to find. Anyway, it knows Java. It knows your default version. It knows the other installed versions. But most importantly for us, you can hand it different filters and get a path to a valid Java home folder back.

Our solution is based around the `java_home` tool. Let’s see it!

## A solution

I keep this in my `.zshrc` file. I have _no_ idea if this will work in Bash or some other shell – it’s untested in everything but Zsh. Sorry 🤷.

```sh
# Extract version from release file in java folder
_java_version() {
    local version_path=$1
    if [[ -f $version_path/release ]]; then
        cat $version_path/release | grep "JAVA_VERSION=" | sed -E 's/JAVA_VERSION=|\"//g'
    else
        echo $path
    fi
}

# List current version with "jdk", or change with "jdk 12"
jdk() {
    local version=${1:-""}
    local silent=${2:-false}
    if [[ $version = "" ]]; then
        java -version
    else
        export JAVA_HOME=$(/usr/libexec/java_home -v"$version");
        if [[ $silent = false ]]; then
            java -version
        fi
    fi
}

# List available versions, global version, and local version with "jdks"
jdks() {
    echo "Java versions"
    /usr/libexec/java_home -V 2>&1 | sed 1d | sed '$d' | cut -d, -f1
    echo "Global: $(_java_version $(/usr/libexec/java_home))"
    if [[ ! -z $JAVA_HOME ]]; then
        echo "Local:  $(_java_version $JAVA_HOME)"
    fi
}
```

The usage is mostly there as a comment above each function, but to explain the workflow: list available versions on the system, along with current global and possibly local version with the command `jdks`. No arguments needed here. To list the active version in your current shell, use the command `jdk`. Use this last command together with a version number to change version locally: `jdk 1.8` gives you Java 1.8, `jdk 12` gives you Java 12. You don’t need to be completely specific, `jdk 1.8` chooses 1.8.0_265 on my system, even though 1.8.0_151 is installed as well. I’m not fluid on the heuristics here, but it mostly seems to behave.

## Bonus

One thing I have added later on is loading a Java version from a `.java-version` file. With the snippet below, if I visit a folder with a `.java-version` file in it, the version is automatically changed. Could it be any easier? Yeah, I guess – it could check recursively, but I haven’t bothered yet. That sounds like it could also slow down my shell as it would happen on every directory change, so there might be a trade off here. As of now, this fills my needs.

```sh
# Put this in your .zshrc file as well

# Automatically change java version from .java-version file
autoload -U add-zsh-hook
_jdk_autoload_hook() {
    if [[ -f .java-version ]]; then
        jdk "$(cat .java-version)" true
    fi
}

add-zsh-hook chpwd _jdk_autoload_hook
```
