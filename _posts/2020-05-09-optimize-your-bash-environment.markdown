---
layout: post
title: "Optimize your bash enviornment"
date: 2020-05-09 23:16:26 +1200
categories: [technical, bash]
tags: [bash, shell]
---

If you are a software engineer or an IT student, chances are you work on terminal shell like bash.
You can perosonalize bash to make yourself efficient and type less to help the lazy guy in you to do his job.

## Type less with aliases

You don't have to know a lot to do this, you just need to know about these two files:

`~/.bashrc and ~/.bash_profile.`

Depending on your operating system you will modify one of these.
On linux machine typically you would use `~/.bashrc` on mac you would use
`~/.bash_profile` .

In rest of text i will assume `~/.bashrc` is the one used by your terminal.

You can create your custom commands and combine multiple commands into one.

for eg. add any or all of the following commands to end of your `~/.bashrc` file.

- If you always work in the same directory u could make an alias, say dev, to go to the directory from any location `alias dev="cd /path/to/dev"`

- You can also write complex functions, for eg. below function combines creating and moving into a directory into one command. After adding below function to your file you could run `mkcd myNewDirectory` to make myNewDirectory

  ```bash
  function mkcd() {
  mkdir $1
  cd $1
  }
  ```

Once you modify any of these files you have to either reopen a new terminal or source that file for eg. `source ~/.bashrc`
### View already defined aliases
In case you forget your custom commands you can list all the aliases active in your terminal using command

```bash
$ alias
```
Above command will not list any functions you defined.

To see a specific alias u can specify its name as the argument

```bash
$ alias dev
```
### View already defined functions
You can see all the functions, without thier definition, using declare command

```bash
declare -F
```
If you want to see definition of functions as well, use 

```bash
declare -f
```

You can also supply the name of specific functions

```bash
declare -f mkcd
```

## Quickly change VIM theme

Just type this in ~/.vimrc file

```
:color desert
```

desert is the color theme here.
