---
title: "Navigation pseudo-mode in tmux"
date: 2023-06-19T09:16:11+01:00

---

I use tmux all the time. Together with NeoVim, it forms a significant part of my workflow in the terminal. A common and valid gripe however is the ergonomics of the prefix key, especially when performing repetitive commands.
However after a little digging on the internet, I learned about key-tables within tmux and a little about how to use them.

My personal pet peeve was the navigation and sizing of panes in tmux. The default bindings are perfectly natural, but even moving my hand to the arrow keys feels like a break of flow sometimes and by contrast I much prefer how I move between splits in vim.
If you've used tmux much at all you're probably aware that the prefix is not mandatory but instead only needed to avoid conflicts - you can bind to `e` if you don't mind never typing words containing 'e' ever again.
You may also be aware of the built-in `copy-mode`s and their key-tables. This is what we will use for our own pseudo-mode. Here's a basic version of my vim-style navigation bindings.

```.tmux.conf
bind -T nav h select-pane -L
bind -T nav l select-pane -R
bind -T nav k select-pane -U
bind -T nav j select-pane -D

bind -T nav H resize-pane -L
bind -T nav L resize-pane -R
bind -T nav K resize-pane -U
bind -T nav J resize-pane -D

bind -N "Enter nav mode" N \
    set prefix None \;\
    set key-table nav \;

bind -N "Exit nav mode" -T nav Escape \
    set -u prefix \;\
    set -u key-table \;
```

The first half defines the key bindings in our soon-to-be mode. We include the `-T nav` argument to push the bindings to a new key-table called "nav".
The last two `bind` calls do the work of switching modes. The first of these lets us enter navigation mode from the normal prefix mode with `<prefix> N` by switching the active key-table. I also disable the prefix by setting it to `None`, but you could equally chose a different prefix.
The final `bind` gives us a way out by reverting the key-table and prefix back to their default bindings.

Now I can hit `<prefix> N`, move around and resize panes with the `hjkl` keys I'm used to in vim, and hit escape to go back to normal.
There's more to be done here - such as adding an indicator to the status line to show if tmux is in `nav` mode or not - but this is a good start.

Thanks to [@samoshkin](https://gist.github.com/samoshkin) for [this great gist](https://gist.github.com/samoshkin/05e65f7f1c9b55d3fc7690b59d678734) that helped me deconstruct how to do this.
For those interested, [here is my full terminal configuration.](https://github.com/mike-deakin/dotfiles)
