---
layout: default
title: Home
nav_order: 1
description: "My personal notebook"
permalink: /
---
## Welcome to my notebook

It's always cool to have a personal space to note down something, and may or may not share it with the world.
Though, in most cases it can be just some mumbo-jumbos coming out of my mouth 😹

### Markdown

I am choosing Markdown to format my notes, because it's a light-weighted and powerful tool to use...
*(nope, because I don't really know anything about html)*

For more details see:

* [Markdown cheatsheet](https://www.markdownguide.org/cheat-sheet/).
* [Some Math symbols (LaTex, Markdown)](http://mohu.org/info/symbols/symbols.htm)

### Jekyll

This Github Page is set up with a Jekyll theme called [Just the Docs](https://just-the-docs.github.io/just-the-docs/). There are still a ton of things I don't quite understand about this Jekyll.
So yeah baby steps, don't rush it!

### Contact

By accessing this page, you already got my Github info, just leave a message!

### Color Scheme

The default scheme is light color, if you prefer a darker scheme, try out the button down below:

<button class="btn js-toggle-dark-mode">Switch to dark scheme</button>

<script> 
    const toggleDarkMode = document.querySelector('.js-toggle-dark-mode'); 
    jtd.addEvent(toggleDarkMode, 'click', function()
    { 
        if (jtd.getTheme() === 'dark') 
        { 
            jtd.setTheme('light'); 
            toggleDarkMode.textContent = 'Switch to dark scheme'; 
        } 
        else 
        { 
            jtd.setTheme('dark'); 
            toggleDarkMode.textContent = 'Switch to light scheme'; 
        } 
    }); 
</script>
