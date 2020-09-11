---
layout: post
title:  "Playing with Gems"
date:   2020-09-06 23:31:00 -0700
categories: jekyll update
background: '/images/fog.jpg'
---

Today, I decided to teach myself some web development skills. I've made basic websites in the past with `html` & `css`, but always bumped up against certain limits.
Making a blog with post previews, for example, seemed like a daunting exercise in repetition & coordination of multiple edited files per post — or at best, an interesting but exceedingly involved exercise in scripting.
I was familiar enough with `jekyll serve` to know these type of problems must already have solutions, so it felt like time to familiarize myself with some new tools.

The tools we're learning about today are:

* Jekyll
* Bootstrap
* rbenv
* Bundler

## Jekyll

Jekyll is a powerful tool for building websites, but to be honest, I'm still just beginning to grasp what it can do. From the [readme](https://github.com/jekyll/jekyll/),

> "Jekyll is a simple, blog-aware, static site generator perfect for personal, project, or organization sites.

When I tackled my first `html` tutorial over at [Learn Enough to Be Dangerous](https://www.learnenough.com/html) (back when it was free), I was taught that Jekyll offers a handy way to test out your website locally.
After going through the rigmarole of installation, we can type up a quick `hello world`,

```bash
$ echo 'hello world' > index.html
```

and all it takes is

```bash
$ jekyll serve
```

to see your mockup in-browser at the local address `http://127.168.0.1:4000`. The first time that worked, it felt pretty special!

Another handy trick [Learn Enough](https://www.learnenough.com/) taught was the basic use of layouts. 
Using some simple markup, Jekyll offers us the ability to extract (say) our website header into its own `header.html` file. 
Then we can include that snippet of `html` at the beginning of multiple site links. 
The snippets get assembled during `jekyll serve` (locally) or `jekyll build` (when publishing online) and we see the conglomerated effect of them in our browser.

This aspect of Jekyll really caught my interest. It felt like a piece of the "blog preview" puzzle. But I had other projects on the go at the time, so now, we're checking it out.

## Bootstrap

Next on my journey, I ran into Bootstrap — an entire library of standardized css classes. The initial appeal to me, was the option to focus solely on the `html` file, rather than continuously referring back to & editing my stylesheet as well.

However, Bootstrap does much more than that! 
The entire library is built to be responsive to the size of your view-screen, with priority focus on the mobile user. 
By using Bootstrap element classes, our webpages should at least be presentable on smartphones, tablets, and large desktop monitors alike, without wasting our time nitpicking around `em`s and `vw`s.
For hobbyists like me with limited access to testing devices, this is a big plus.

To apply Bootstrap to your web project, include these stylesheets in your page header:

```html
<!-- Latest compiled and minified CSS -->
<link rel="stylesheet" href="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/css/bootstrap.min.css">

<!-- jQuery library -->
<script src="https://ajax.googleapis.com/ajax/libs/jquery/3.5.1/jquery.min.js"></script>

<!-- Popper JS -->
<script src="https://cdnjs.cloudflare.com/ajax/libs/popper.js/1.16.0/umd/popper.min.js"></script>

<!-- Latest compiled JavaScript -->
<script src="https://maxcdn.bootstrapcdn.com/bootstrap/4.5.2/js/bootstrap.min.js"></script>
```

Check out the [w3s tutorial](https://www.w3schools.com/bootstrap4/bootstrap_get_started.asp) to get a feel for some Bootstrap basics.

At the end of the day, Bootstrap takes care of a lot in terms of portability, but it still leaves us doing most of the legwork in terms of templating, and it sure doesn't solve the problem of building an easily updatable blog site.

Happily, many users have shared their own Bootstrap-based templates & themes! If you really just want to get a website up and running, you can check them out [here](https://startbootstrap.com/themes/). Of course, for the curious, that's not a very satisfying ending.

I did find an [interesting Bootstrap theme](https://startbootstrap.com/themes/clean-blog-jekyll/) that incorporates Jekyll into its templating.
Once we have a bit more understanding of how each piece works, we'll test it out.

## rbenv

Jekyll is written in Ruby & distributed as a Ruby 'gem', aka. a portable self-contained piece of code. 
In order to make our web-dev environment truly reproducible, we need to track which version of Ruby we're developing with. 
`rbenv` fills this role, in the same way that `virtualenv` does for a Python environment.

To install `rbenv`, we can either grab it with Homebrew on Mac (`brew install rbenv`) or `git clone` from [source](https://github.com/rbenv/rbenv).

`rbenv` manages your Ruby versions by adding shims to your system `$PATH`. 
Whenever we enter Ruby commands such as `jekyll` or `bundle`, they are intercepted & rerouted to the appropriate binary.

Using `rbenv` is simple. To set up a global default Ruby version,

```bash
$ rbenv global 2.7.0
```

(This is saved in `~/.rbenv/version`.) Or, to set up a local Ruby environment,

```bash
$ rbenv local 2.7.0
```

which creates a `.ruby-version` file in your project home.

#### A few notes:

* `rbenv versions` checks which ruby versions are on your machine

```bash
$ rbenv versions
  system
  2.3.3
  2.7.0
* 2.7.1 (set by ~/playing_with_gems/.ruby-version)
  2.7.1_2
```

* `rbenv install -l` lists all stable ruby releases

```bash
$ rbenv install -l
2.5.8
2.6.6
2.7.1
jruby-9.2.13.0
maglev-1.0.0
mruby-2.1.2
rbx-5.0
truffleruby-20.2.0
truffleruby+graalvm-20.2.0
```

* and of course, `rbenv install VERSION` installs a version locally.

## Bundler

With our Ruby environment set up, our next task is managing versions for our gems, which is where Bundler comes in.

Bundler tracks gem dependencies. Although `gem` is the Ruby package manager, we will be sending all our install requests to `gem` via Bundler. 
We will also execute any gem commands (like `jekyll serve`) via Bundler using the `bundle exec` prefix. 
Along with `rbenv`, Bundler ensures a reproducible Ruby environment.

Install Bundler with `gem`:

```bash
$ gem install bundler
```

And record gem versions with a file called `Gemfile` (or let Bundler add them for you).

___

# The Essentials

Say we want to set up an environment to develop a website using Jekyll, and maybe add some other gems along the way.

### Git

First, of course, set up version control:

```bash
$ git init
```

### rbenv

Then, use `rbenv` to declare which version of Ruby to use.

```bash
$ rbenv version 2.7.1
```

### Bundler

Next, initialize our `Gemfile`.

```bash
$ bundle init
```

Optionally, declare a directory to hold installed gems.

```bash
$ bundle config set --local path 'vendor/bundle'
```

When we `bundle add` gems to our `Gemfile`, Bundler auto-installs dependencies as well.

### Jekyll

To use Jekyll in our project,

```bash
$ bundle add jekyll
```

If that works, great! However, in my case I had to wrestle with a rather opaque issue. 
One dependency of Jekyll, `http-parser.gm`, will throw install errors via Bundler if your working directory has any *spaces* in its address. 
Aside from the obvious workaround, see [here](https://github.com/tmm1/http_parser.rb/issues/47) for other ideas.

Once Jekyll is added succesfully, we create a Jekyll scaffold for our site. Remember to wrap Jekyll commands within `bundle exec` like so:

```bash
$ bundle exec jekyll new --force --skip-bundle .
$ bundle install
```

If, like me, you set Bundler to store gems in a local directory like 'bundles', rather than 'vendor/bundle' specifically, then there's one more step before serving. We need to tell Jekyll **not** to scan our gems for blog posts! (Jekyll interprets Markdown files such as READMEs as blog posts which can lead to formatting issues in this case.)

```bash
$ echo "exclude: ['bundle/']" >> _config.yml
```

Alright, with that, it's time to serve!

```bash
$ bundle exec jekyll serve
```

Notably, if you want to view your locally hosted site from other devices (e.g. a smartphone):

```bash
$ bundle exec jekyll serve --host 0.0.0.0
```

Then we can view our site at `192.168.X.Y:4000` for other devices (where X.Y is the host device's local address), and `0.0.0.0:4000` on the host machine.

(Check your local address with `ifconfig | grep 192.168` on Mac.)

My fave Jekyll serve call also auto-refreshes browser tabs when we save changes to hosted files:

```bash
$ bundle exec jekyll serve --livereload --host 0.0.0.0
```

Super handy. :)
