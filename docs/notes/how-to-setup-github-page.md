---
layout: default
title: How to Setup Github Page
parent: Notes
nav_order: 2
---
## How to Setup Github Page

### Prerequistes

There are a few things that need to be done before setting it up.

* Github account & git
* Ruby env (if use [jekyll](http://jekyllrb.com/))

### Github & git

As a developer/programmer, Github should be no stranger, and [git](https://git-scm.com/), is a version control system to version control your site content and sync it with Github in order to publish the changes online.

Besides git, other SCM(Source Control Management) tools are quite heavier than git, such Perforce (which can however handle large-file project).

(the LFS of Github is so limited...😕)

#### Create a Remote Repo

Here are some tips for this repo:

* name has a pattern as: `<your-github-username>.github.io`, to make sure the name is unique

  * seems `.github.com` also works
* in the **Settings**, find the **Pages** subpage

  ![image.png](./assets/1657812228386-image.png)
* (optional) choose a theme by clicking `Choose a theme` or configure it later using Jekyll.

> At first I misspelled my username (hooyay...), so my page got published at ~~https://idioticprogramape.github.io/idiotprogramape.github.io/~~ 👀️

#### Create a Local Repo

This step will require a local repo cloned from the previously created github repo.

There are 2 ways to clone the repo from github:

1. use git CLi (example is SSH, need some more configuration, can use https link as well)

   ```bash
   git clone git@github.com:<your-github-username>/<your-github-username>.github.io.git
   ```
2. use some GUI application such as Github Desktop to avoid CLi.

### Create Jekyll Project

#### Install Jekyll

I'll just copy paste from Jekyll's website here, this only works on **Windows**, for other OS, need to check these [Guides](https://jekyllrb.com/docs/installation/#guides):

1. Download and install a **Ruby+Devkit** version from [RubyInstaller Downloads](https://rubyinstaller.org/downloads/). Use default options for installation.
2. Run the `ridk install` step on the last stage of the installation wizard. This is needed for installing gems with native extensions. You can find additional information regarding this in the [RubyInstaller Documentation](https://github.com/oneclick/rubyinstaller2#using-the-installer-on-a-target-system). From the options choose `MSYS2 and MINGW development tool chain`.
3. Open a new command prompt window from the start menu, so that changes to the `PATH` environment variable becomes effective. Install Jekyll and Bundler using `gem install jekyll bundler`
4. Check if Jekyll has been installed properly: `jekyll -v`

#### Initialize Jekyll site

1. open a terminal, and **cd** to the local repository
2. set up a jekyll project by exec: `jekyll new .`
3. add `webrick` to the dependencies by exec: `bundle add webrick`
4. build the site: `bundle exec jekyll serve --incremental --livereload`
5. check [http://localhost:4000](http://localhost:4000/)

#### Apply Theme

Since the framework of a Jekyll site has been built, now need to choose a theme and apply it.

##### Install Theme

The themes provided on Github settings are very limited, here are some links where can find more themes:

* [GitHub.com #jekyll-theme repos](https://github.com/topics/jekyll-theme)
* [jamstackthemes.dev](https://jamstackthemes.dev/ssg/jekyll/)
* [jekyllthemes.org](http://jekyllthemes.org/)
* [jekyllthemes.io](https://jekyllthemes.io/)

Finally I chose the simple theme called [just-the-docs](https://github.com/just-the-docs/just-the-docs) to start with. Normally the installation is well documented, just need to check the guide.

> Remarks:
>
> * In the file **_config.yaml**, there are 2 keywords for theme:
>   * **remote_theme**: this one is for github, so that the online site can be built with the correct theme
>   * **theme**: this one is for offline debugging.
> * After search data initialization(to enable search functionality), a json file was generated at: *assets/js/zzzz-search-data.json*
>   * this file name is not consistent with its **permalink**, need to unify them
>   * a small syntax error to fix:
>
>     ```diff
>     - {**%- assign pages_array =  | split:  -%**}
>     + {**%- assign pages_array =  split:  -%**}
>     ```

#### Configure Theme

For my theme, they provided some specific params described in [here](https://just-the-docs.github.io/just-the-docs/docs/configuration/).

It should be the same for other themes, if not, use _config.yaml from their Github repo as a start is not a bad choice.

### More...

So far, after doing the steps above I can already have the very first glimp of my page.

Later on when need to add more sections and pages, here is a ref: [Navigation Structure](https://just-the-docs.github.io/just-the-docs/docs/navigation-structure/).
