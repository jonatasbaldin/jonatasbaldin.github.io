---
layout: post
title: "New experimental setup (text editor and stuff)"
date: 2016-09-27 10:00:00
tags: ['setup', 'neovim', 'zsh', 'oh-my-zsh', 'hypertem', 'nova']
description: "Today I will take a break from containers and write a little bit about how I get stuff from my head into zeros and ones. I've been using open source operating systems - a.k.a. Linux - for 6 years and last month I got a Macbook Pro from work and once again I went into the wild searching for awesome stuff to run on OS X :sunglasses:"
comments: True
---

Today I will take a break from containers and write a little bit about how I get stuff from my head into zeros and ones. 

I've been using Linux for 6 years and last month I got a Macbook Pro from work and once again I went into the wild searching for awesome stuff to run on OS X :sunglasses:

Before I forget, now I'm a Python developer, so you may see some fine tuning for it.

# I can't drop vim, I just can't
![neovim](/img/setup_neovim.png){: .center-image }

Unfortunately (or fortunately) I'm not ready for an IDE. [Vim](http://www.vim.org/) has been my text world since I started with Linux and it looks like it will be for a long time. I'm used to its navigation, motions and commands.

Now I'm trying out vim's refactor, [neovim](https://neovim.io/). It has some cool stuff, like a terminal simulator, true color and a nice API.    

# Thank you for everything, bash
<br>
![ohmyzsh](/img/setup_ohmyzsh.png){: .center-image }
<br>
Oh, [bash](https://www.gnu.org/software/bash/), you taught me so much! Commands, redirectors, pipes, flags, returns, scripting. You did your job and you did it well, but now the new cool kid has arrived at my terminal: [ZSH](http://www.zsh.org/) "Z Shell". 

It is awesome! And boosted with [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) it gets extremely powerful. Just two days using it and I'm in love. Nice tab completion, themes and plugins. I just run `z <one-or-few-chars> <tab>` and it gets me anywhere I want. Neat.

This one is here to stay. Strongly recommended!

# Multiplex Everything
![tmux](/img/setup_tmux.png){: .center-image }
[Tmux](https://tmux.github.io/) is THE gold mine. Once you get it, you never leave it. 

Basically it muliplies your terminal in panes, tabs and windows. You can navigate between them seamlessly, copy, paste, search, attach, detach and do a lot. It does everything you may think of.

Bonus point: [vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator) allows you to switch between vim and tmux panes. I love this plugin. I would marry it.

# A Javascript Terminal
![hyperterm](/img/setup_hyperterm.png){: .center-image }
Nowadays you can run a web based terminal. [Hyperterm](https://hyperterm.org/) is where I'm typing this article right now. I still have doubts about this guy, but I'm giving it a try. The `hypermode` is fun. You can browse inside the terminal, and configuring it is pretty simple.

# Nice new colors
[Nova](http://www.trevordmiller.com/nova/) is the new Solarized. Amazing color scheme, you get used to it really fast and stop liking anything else. The colors have real meaning and are comfortable. It integrates well with neovim, tmux and Hyperterm, so it was an obvious choice. You're gonna love it!

---

**YOU CAN HAVE EVERYTHING :relieved:**

I store all my configurations in [this GitHub repository](https://github.com/jonatasbaldin/dotfiles). 

It has a poor installation script (if you want to, you may refactor it for me...), but you can investigate the configurations and try them yourself. I suggest that, if you are going to do that, try it line by line, reading the software documentation. Don't just copy and paste it. Productivity comes when your tools does what *you* need. Also, it is a fun process :smile:

There you go, an overview on how I work. Of course I use other tools, like Git, Slack, Spotify, Evernote, Postman and so on, but the core business is here. Leave a comment if you have any feedback, and share the knowledge!

*This post was revised by my friend [Raphael Hernandes](https://raphaelhernandes.com/). Thanks man!*
