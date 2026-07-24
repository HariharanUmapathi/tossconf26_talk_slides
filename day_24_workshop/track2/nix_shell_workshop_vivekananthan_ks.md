---
theme:
  name: catppuccin-mocha
options:
  end_slide_shorthand: true
  implicit_slide_ends: false
---
<!-- jump_to_middle -->
Thought for the day:

"Following existing established ways is one way to solve,
**And so is finding a way**. That's the birth of innovation"
===
<!-- font_size: 7 -->
<!-- end_slide -->
<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

![image:width:50%](nixlogo.png)
<!-- column: 1 -->
**Adopting Nix from Day 1 through nix shells for absolute beginners**
===
<!-- reset_layout -->

<!-- new_lines: 3 -->

<!-- alignment: center -->
**By**

## Vivekanandan KS

![image:width:80%](profilepic.jpg)

---

<!-- font_size: 7 -->
# What to Expect
<!-- incremental_lists: true -->
* **Target Audience: Absolute beginners to veterans**
* **Core Goal: Start using Nix shells for your everyday workflows right away**
* **First Principle Thinking: Pure functional declarative vs. imperative automation**
* **Bonus (if time permits): Introduction to `devenv` (a beginner-friendly, lightweight wrapper for dev environments, an alternative to dev containers)**

<!-- font_size: 7 -->

# Expected Outcomes


**By the end of this practical session, you'll gain a new mental model for approaching modern workflows.**

<!-- incremental_lists: true -->
* **Understand: What Nix is and its broader ecosystem**
* **Apply: How to use Nix shells and `nixpkgs` from day 1**
* **Compare: Why a pure functional declarative approach is far more reliable and necessary compared to imperative setups**
* **Decide: Why containers (Docker, Podman) aren't always the necessary solution, and when/where to choose reproducibility and isolation instead**

---

<!-- font_size: 4 -->

<!-- column_layout: [1, 1] -->
<!-- column: 0 -->

## Pre-requisites

**You should have Nix running on your system!**
**(Linux, macOS, or Windows via WSL)**

**Install from here:**

https://zero-to-nix.com/start/install/#up

__And some curiosity, interest etc :-)__

<!-- column: 1 -->

## Format

**This is a practical, hands-on session.**

**So feel free ask any doubt if u think it's stupid**

<!-- reset_layout -->

---

<!-- font_size: 4 -->

# Let's Dive In!

---

**Question:**

Why organisation restrict their employees from installing external packages in their work system?

<!-- pause -->
**Answer:**
<!-- pause -->
“It works on my machine”
Installing dependencies pollute the user space.
Even when u install dependencies in a pre-Script phase, the post-Script phase cleaning process should be careful to avoid polluting the user space again.
Imagine midway script failure during any of these phases. Manual Rollbacks and tinkering becomes a maintenance overhead.
Maintain dev, test, prod separately and continuously tinker to sync it.

---
Nix Shells
===
Temporarily expose the package from /nix/store to the environment.

Temporary Expose =  Adding the package binaries to the PATH temporarily

A very neat and simple trick.
Multi version coexistence of the same package isn't at all a problem anymore.

---
**Today we gonna learn few different types of usage of Nix shells. So that u can start using them directly in ur everyday workflows**

Nix shells usage types:
===

Scope for today:
# Ad-Hoc Shells & One-shot method

I'll give an overview depending on the time:
# In a script file
# Scripting inside flakes(portable)
# devenv

---

Let's start doing now
===
---
Nix Ad hoc shell environments:
===
Let’s do some simple commands:

```bash
echo “hello world” | cowsay
```

But cowsay doesn’t exist. So let’s enter a nix shell with the cowsay package,

```bash
nix-shell -p cowsay
```
Now try the command again:
```bash
echo “hello world” | cowsay
```
---

What’s happening?
===

# Nix built the packages mentioned

# Nix created a temporary shell environment with the built packages’ binaries added to the PATH.

Now exit the shell by typing `exit` and try the hello world command again.

U can see that Nix removed the binaries from PATH without polluting user environment.

---

Now try this command
===
```bash
echo "hello, world" | cowsay | lolcat
```

Solution in next slide...

---

Solution
===

```bash
nix-shell -p cowsay lolcat
```

you enter the shell and now run:
```bash
echo "hello, world" | cowsay | lolcat
```

**We can also launch a shell and run command in one go too:**
 ```bash
 nix-shell -p cowsay --run '<actual-command-here>'
 ```

 Eg:
 
 ```bash
 nix-shell -p cowsay --run \
'echo "hello, world" | cowsay | lolcat'
 ```
---

You can also use the --command / -c too
The Core Difference

`--run`: Executes the command in a `non-interactive shell` and then immediately exits back to your original terminal.

`--command`: Executes the command in an `interactive shell`. It sources interactive shell initialization files (like .bashrc or .zshrc). 

**Tip:add return after the command if u dont want to exit the interactive shell**

Try this:
```bash
 nix-shell -p cowsay --run \
'echo "hello, world" | cowsay | lolcat ; return'
```
For our simple example both flags behave the same it behaves the same.

---
Some other basic flags:
===
`--pure`

If  this  flag  is  specified, the environment is almost entirely cleared before the interactive shell is started, so you get an environment that more closely corresponds to the “real” Nix build. A few variables, in particular HOME, USER and DISPLAY, are retained.

`--keep`

When a --pure shell is started, keep the listed environment variables.

`--impure`

Allow access to mutable paths and repositories. Useful to pass environment variables to a command

Run the good old --help flag to dive deep
---
```bash
nix-shell --help
```
---
nix run
===

Directly run the program directly without installing in the user environment just like nix-shell but different use cases

U can do this🔥🔥:
```bash
nix run nixpkgs#cowsay -- "hello, world"

```
Which is equivalent to the normal command:
```bash
cowsay "hello, world"
```
Everything after the "`--`" are just usual flags and arguments usage of the program

---
U can also use the nix run command as if it's a package in your system
===

```bash
echo “hello, world” | nix run nixpkgs#cowsay
```

which is equivalent to running:
```bash
echo “hello, world” | cowsay
```
---

Now Try this:
===
# Make this command adhoc with Nix:
```bash
echo “hello, world” | cowsay | lolcat
```

# Answer:
```bash
echo "hello, world" | nix run nixpkgs#cowsay | nix run nixpkgs#lolcat
```
 which can also be done like:

```bash
nix-shell -p cowsay lolcat --run ‘echo “hello, world” | cowsay | lolcat’
```
---

Solve this case:
===

# Case 1:

You want to learn to use a particular program named `hello` and u know u can do it with the --help flag. And since the --help output is just monochrome u find it hard to read. So upon exploring u learnt that a program named bat can help colorify the output for better readability. But programs are not installed in ur system. Since it's your office laptop, you dont want to install things too and dont wanna ovcerkill with docker etc

The commands are:
```bash
hello --help
bat -l help 
# -l help colorify properly for --help flags
```
Now how can u try this without installing ?

<!-- pause -->

# Case 2
<!-- pause -->
Now I have one or both packages installed in my system, but I wanna demonstrate this to you all. How should I approach without uninstalling anything and without the need to create and launch a container environment ?

There's no right or wrong solution to this. But let's see how u all solve this

---

Congratulations on solving it

---
Towards more reproducibility and determinism
===

```bash
nix-shell -p cowsay lolcat \
 --pure \
 -I nixpkgs=<NIXPKGS_REFERENCE> \
 --run 'echo "hello, world" | cowsay | lolcat'
```
where <NIXPKGS_REFERENCE> can be:
 * nixpkgs-unstable (rolling release channel)
 * nixpkgs-26.05 (stable release channel)
  * nixpkgs-25.11 (previous stable channel)
 * https://github.com/NixOS/nixpkgs/archive/refs/heads/nixos-25.05.tar.gz
 * https://github.com/NixOS/nixpkgs/archive/dad564433178067be1fbdfcce23b546254b6d641.tar.gz

 etc

---
<!-- jump_to_middle -->
Any Doubts so far🙋?
===

---
Reproducible scripts using Nix
===

Use the fundamentals so far to level up your scripting

Imagine a simple script like this:
```bash
#!/usr/bin/env bash
echo "Hello, myself!" | figlet | lolcat
echo "You're a great person!" | cowsay | lolcat
python --version
```
# Qn) 
What is wrong with this script? What if others run this?

<!-- pause -->
# Ans: 
One of the Ans) It assumes the system have installed the dependencies like python, lolcat, cowsay and figlet.

---

Reproducible interpreted scripts with Nix
===

```bash
#!/usr/bin/env nix-shell
#! nix-shell -i bash --pure --quiet
#! nix-shell -p bash cowsay figlet lolcat python312 nix
#! nix-shell -I nixpkgs=channel:nixos-25.05
echo "Hello, myself!" | figlet | lolcat
echo "You're a great person!" | cowsay | lolcat
```
It’s the same script with nothing changed in the body
But why there are multiple shebangs? What's going on? Any Guess?

<!-- pause -->
The first line is the usual shebang mentioning the interpreter to use which is nix-shell in this case.
```bash
#!/usr/bin/env nix-shell
```
And the following shebang like lines are interpreted by nix-shell.
Instead of giving out the nix-shell command with all flags in a lengthy format, for readability we are splitting it into multiple lines.
```sh
#! nix-shell -i bash --pure --quiet
#! nix-shell -p bash cowsay figlet lolcat python312 nix
#! nix-shell -I nixpkgs=channel:nixos-25.05
```
---

# Are we nesting the shell with different flags? That will start unnecessary child processes and not efficient right? Haha

Actually the nix-shell interpreter combines all those flags and options and start a single environment instead of nesting. As u might guess the shebang like syntax is the signal for it to combine the flags and options.

Now with just adding these 3 more lines at the top without even disturbing the body we have just made this script 100% deterministic to work on any system.
See we didn’t even expect bash to be present, Nix is the only dependency we need. 

Super cool right? :-)

---
Let’s break down the new flags:
---
`#! nix-shell -i bash --pure --quiet`


`--quiet` : Just avoid the verbose output while building the dependencies

`-i bash` : Chooses the interpreter to use for the of the script


So basically with nix-shell we create the environment to run a command and with nix reproducible scripting we set the environment at the start of the script so that the script can use those apps.

U can even take it further by using nix run commands inside the scripting so that u can run a program without even polluting the script environment.

---
Nix is a vast ecosystem at this point. U just saw the usage of nix-shells, nix run commands and their flags and there are a lot of super cool projects which revolutionizes tons of things in modern software industry. Explore at your pace.
===
The next step is for u all to try devenv to manage dependencies for ur project , vendor agnostic secrets management, services etc etc. 

---

Some nix ecosystem projects:
===
1) `NixOS` - declaratively linux and Framework
2) `Home-manager` - dotfiles management
3) `nixDockerTools` - dockerFile replacement
4) `devenv` - nix shells
5) `nix-wrapper-modules` - create preconfigured distros for ur programs like neovim, etc etc
6) `system-manager` - ansible replacement for non NixOS distros
etc etc 
7) `NixBSD` - NixOS type framework but for FreeBSD
google awesome-nix github and you'll find really cool projects
8) `nix-build` - World's most advance build system which is universal for creating literally anything from nix shells, nix run, nixpkgs, nixos, containers etc
---
<!-- jump_to_middle -->
Congratulations for making to this point. Now u can start using nixpkgs through nix-shell and nix run commands without even installing in your system. Now go start trying things without the fear of breaking your system.
===

---
<!-- jump_to_middle -->
Thank You
===

Vivekanandan KS (vivek)
===
---
