---
layout: post
title: "Projects: QuickStart.sh"
date: 2022-10-16 18:22 +0100
tags: projects debian ubuntu bash
categories: [Projects, Automation]
published: true
---

I got tired of doing the same things everytime I setup a VPS or VM, so I wrote a little script to automate the installation process of a few things.

[QuickStart.sh](https://github.com/bull-sec/QuickStart)

There's nothing crazy really going on in any of this, it's just a setup that I've built up over the years and I find that it gives me all of the basics to use a Debian-based machine as if it was my daily driver.

![Custom Bash + TMUX + FZF + Bat](/assets/img/2022-10-16-13-33-54.png)

## Custom Bash Prompt

## Customised TMUX Configuration

As you can see from the screenshot, it's purple.

The TMUX default key has also been switched from `Ctrl+B` to `Ctrl+A`, three reasons for this.

1. I prefer it
2. `screen` uses `Ctrl+A` as it's default keybind, so I don't need to remember two
3. If comes in handy when I end up on a box and need to spawn a TMUX session inside my TMUX session

![tmux inside tmux](/assets/img/2022-10-16-13-43-05.png)

![yo dawg](/assets/img/2022-10-16-13-40-02.png)

## FZF

FZF is just awesome, I mostly use it as a replacement for the "reverse-search" (Shortcut: `Ctrl+R`) function for my Bash history, but generally it's just pretty handy to have a "Fuzzy Search" tool available when looking for files and lines in code.

![FZF in "reverse-search" Mode](/assets/img/2022-10-16-13-45-37.png)

## BAT

Bat is a replacement for good old dependable `cat`.

The difference between the two is Bat has a built in paging function, and it also does syntax highlighting.

Using it in conjunction with FZF as a file preview is magical.

```bash
# Create the following Bash Alias
alias preview="fzf --preview 'bat --style=numbers --color=always --line-range :500 {}'"

# Run it via vim
vim $(preview)
```

![FZF + Bat = Bliss](/assets/img/2022-10-16-13-50-04.png)
