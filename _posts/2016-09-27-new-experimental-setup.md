---
layout: post
title: "New experimental setup (text editor and stuff)"
date: 2016-09-27 10:00:00
tags: ['setup', 'neovim', 'zsh', 'oh-my-zsh', 'hypertem', 'nova']
description: "Today I will take a break on containers and talk a little bit about how do I get stuff from my head into zeros and ones. I've been using open source operating systems - a.k.a. Linux - for 6 years and last month I got a Macbook Pro from work and once again I went into the wild searching for awesome stuff to run on OS X."
comments: True
---

Today I will take a break on containers and talk a little bit about how do I get stuff from my head into zeros and ones. I've been using open source operating systems - a.k.a. Linux - for 6 years and last month I got a Macbook Pro from work and once again I went into the wild searching for awesome stuff to run on OS X :sunglasses:

Before I forget, now I'm a Python developer, so you may see some fine tuning for it.

# I can't drop vim, I just can't.
![neovim](/img/setup_neovim.png){: .center-image }

Hey, I'm a developer now, let's see what these IDEs can give me.

Lot of buttons. Lot of borders. Lot of menus. Lot of stuff.

Unfortunately (or fortunately) I'm not ready for an IDE. [vim](http://www.vim.org/) has been my text world since I started with Linux and it looks like it will be for a long time. I'm used to the navigation, motions, commands. They are burned into my mind! My fingers only know how to reach stuff by hjkl.    

But it is not all flowers, vim has its perks now and then, but we always find a balance to be friends. Now I'm trying out [neovim](https://neovim.io/), but did not go to far besides installing and applying my `.vimrc`. It appears to have some cool stuff, like a terminal simulator, true color and a nice API.    

Once I have a concrete opinion, I'll write about it.

# Thank you for everything bash
<br>
![ohmyzsh](/img/setup_ohmyzsh.png){: .center-image }
<br>
Oh [bash](https://www.gnu.org/software/bash/), you taught so much! Commands, redirectors, pipes, flags, returns, scripting. You did your job and you did it well, but now the new cool kid arrived at my terminal. [ZSH](http://www.zsh.org/) "Z Shell" is fucking awesome! And boosted with [oh-my-zsh](https://github.com/robbyrussell/oh-my-zsh) it gets extremely powerful. Just two days using you and I'm in love. Nice tab completion, themes and plugins. I just run `z <one-or-few-chars> <tab>` and it gets where I want, neat.

This one is here to stay. Strongly recommended!

# Multiplex ALL THE THINGS
![tmux](/img/setup_tmux.png){: .center-image }
[tmux](https://tmux.github.io/) is THE gold mine. Once you get it, you never leave it. Basically it multiply your terminal in panes, tabs and windows. You can navigate between them seamlessly, copy and paste, search, attach and detatch and do a lot. If you think it should do something, it does and you just have to find out how.

Bonus point: [vim-tmux-navigator](https://github.com/christoomey/vim-tmux-navigator) allows you to switch between vim and tmux panes. I love this plugin. I would marry it.

# Javascript terminal..........
![hyperterm](/img/setup_hyperterm.png){: .center-image }
Times like these, when you need Javascript to run a terminal. [Hyperterm](https://hyperterm.org/) is where I'm typing this article right now. I still have doubts about this guy, but I'm giving it a try. The `hypermode` is fun, you can browse inside the terminal, and configuring it is pretty simple. Not too much words, just waiting and watching.

# Nice new colors
[Nova](http://www.trevordmiller.com/nova/) is the new Solarized. Amazing color scheme, you get used to it really fast and stop liking anything else. The colors have real meaning and are comfortable. It integrates well with neovim, tmux and Hyperterm, so it was an obvious choice. You gonna love it!

---

**I WANT EVERYTHING :scream:** 

**YOU CAN HAVE IT :relieved:**

I store all my configurations in [this GitHub repository](https://github.com/jonatasbaldin/dotfiles). It has a poor installation script (if want refactor it for me...) but you can investigate the configurations and try yourself. I suggest that if you are gonna do that, try it line by line, reading the software documentation. Does not just copy and paste it, the productivity comes when your tools does what *you* need. Also, is a fun process :smile:

There you go, an overview on how I work. Of course I use other tools, like Git, Slack, Spotify, Evernote, Postman and so on, but the core business is here. Leave a comment if you have any feedback, and share the knowledge!
