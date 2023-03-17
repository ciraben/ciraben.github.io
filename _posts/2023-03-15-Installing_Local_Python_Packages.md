---
layout: post
title:  "Installing Local Python Packages"
date:   2023-03-17 12:00:00 -0700
background: '/images/beach.jpg'
---

Recently, I stumbled across a weird conundrum when playing with Python code. I've been working on a number of small Python projects recently, and while working on a problem, it occurred to me – "boy, it'd sure be handy if I used that function from another project to accomplish this!"

After all, isn't it a ubiquitous phrase in programming to "never solve the same problem twice" or so? If I can learn to re-use my code from other projects, then I'll save time! I might even learn something about writing more easily repurposable code in the process.

At first, I thought - this will be easy! I can just `import` the relevant file from my other project and be done with it. But it's not quite so simple. As it turns out, `import` only really works for local modules contained within a subdirectory of your current working directory. If your file structure is anything like mine, `import`ing modules between projects like this is a no-go.

```zsh
.
└── my_python_projects
    ├── project_a
    |   └── the_module_i_wanna_import.py
    └── project_b
        └-- the_script_i_wanna_import_it_into.py

```

I know that, at the very least, I could *manually copy* my `project_a` code into `project_b`, and for some hobbyists this is fine! But to me, this solution feels cumbersome, and I know there must be something better out there!

I also know that, at most, if we wanted to go all out, we could publish our code as a Python package on **[PyPi](https://pypi.org/)** – then we could simply `pip install` the package whenever. However, this must be more work than necessary, right? Also, what about polluting PyPi's package namespace? Package names can only be used once there, after all – that would be kinda selfish...

To summarize my goal here, I'm looking for the simplest approach that:
- lets me import `.py` files from `project_a` into `project_b`
- provides an easy, memorable way to update my reference to `project_a` code without leaving `project_b`
- minimizes polluting either local workspace with additional files/dirs
- is as non-hacky a solution as possible
After all, I want to build a workflow that I can use a lot going forward, so our solution should be simple, intuitive, & unobtrusive!

# `pip` to the rescue!

As it turns out, `pip` provides built-in functionality for installing a local project as a package! The basic use is:

```zsh
$ pip install ../project_a
```

However, to make this work, we need to add some additional structure to `project_a` first, so that it *"looks"* like a package to `pip`. Specifically, we need to add a `pyproject.toml` file (or the archaic `setup.py` file – **[learn more here](https://bernat.tech/posts/pep-517-518/)**) to our `project_a` directory.

Additionally, we need our `.toml` to contain the following three lines:

```toml
[build-system]
requires = ["yourbuildsystemofchoice"]
build-backend = "yourbuildsystemofchoice.building.module"
```

Here, we define a "build system" for `pip`, which tells `pip` which tool to use to build a local version of our project when running `pip install`. Some options are `setuptools` (Python's default recommend), `distutils`  (archaic), `poetry` (hipster start-up vibes), maybe others!

The tool that we choose determines what other structural changes we need to make to `project_a` in order to convince `pip` that it's a package. For example, `setuptools` likes seeing a `setup.cfg` file in your `project_a` directory (though it's not required).

# The problem with `setuptools`

On the surface, `setuptools` looks like a great option for our minimalist goal! To use, the basic requirement is to make your `.toml` look like this:

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
[project]
name = 'project_name'
version = '0.0.1'
```

And technically, even this still builds – a minimalist's dream!

```toml
[build-system]
requires = ["setuptools"]
build-backend = "setuptools.build_meta"
```

>Note – With the minimal solution above, if exactly one `.py` file is found in your `project_a` folder, `setuptools` gives your package its name, and `version=0.0.0`. If no `.py` file is found (or if they're hidden in subfolders) `setuptools` makes a package called `UNKNOWN` instead, `version=0.0.0`. In both cases, you can `import path.to.script` in Python, where `path.to.script` is the relative path from within `project_a`.
>
>However, if two or more `.py` files sit in your `project_a` folder (instead of in subfolders), `pip install` will fail. This kinda setup needs a more fleshed out `.toml` file.

Despite this wonderful simplicity, we run into trouble when we successfully `pip install`. You see, ever since the release of `pip` **version 21.3** on October 11th, 2021, `pip` switched to what they call "in-tree builds" as a way to save time & space when installing local packages.

The `setuptools` project hasn't adjusted to this new workflow yet. As a result, if we use `setuptools` as our build system of choice (as in the `.toml` above), `pip install` pollutes your `project_a` directory with two directories called `build` and `project_a.egg-info` whenever `pip install` is used.

Additionally, these new files/dirs hang around kinda like a cache, so if you make changes to `project_a`, you also need to remember to delete `project_a/build` in order to force `pip install` to re-build, so that your changes are incorporated. Otherwise, the cached build files are re-used, and you may not notice that your changes to `project_a` are missing from `project_b` even after a reinstall! Yuck.

People are hot for change in **[this Github issue here](https://github.com/pypa/setuptools/issues/3236)**, but until something is implemented, I'm steering clear of `setuptools` for our purposes.

# Building with `poetry`

Since my research shows that `setuptools` (and not `pip`) is the source of the mess described above, what about an alternative? Poetry does have stricter requirements, but let's check it out!

First, we need a more detailed `.toml` file – Poetry requires **all** the following fields:

```toml
[build-system]
requires = ["poetry-core"]
build-backend = "poetry.core.masonry.api"

[tool.poetry]
name = "project_a"
version = "0.0.1"
description = ""
authors = ["test-name <test@name.com>"]
```

>Technically, `pip install` will complete as long as `authors` contains at least one string, and that string is of the form `"a <b>"`, where `a` and `b` are non-whitespace strings themselves. Something like the following works fine!

```toml
authors = ["- <->"]
```

Additionally, `poetry` prefers that `project_a` employ a more package-like directory structure (covered below). However, if your `.py` files are just hanging out in your `project_a` directory like in our example above, you can add the following line to your `.toml` instead:

```toml
[tool.poetry]
packages = [{include = "the_module_i_wanna_import.py"}]
```

... And that's it! With these requirements in place, you can run `pip install ../project_a` in your `project_b` folder, and successfully `import the_module_i_wanna_import` in Python – *without* dumping anything into your `project_a` folder as a side-effect!

Keep in mind that `pip install` still builds a local snapshot of `project_a` for use with `project_b` – so if you want to carry over `project_a` changes, you do need to `pip install ../project_a` again to rebuild & incorporate those changes into `project_b`.

# Keeping it simple

To populate the required `.toml` with the details `poetry` needs:

```zsh
$ poetry init
```

...within your `project_a` directory. This provides you an interactive prompt where you're prompted for `name`, `version`, etc. Mostly, it's super handy & easy to remember!

We've basically reached our goal, but with one caveat. `poetry init` doesn't prompt you to customize your `packages` values – it just adds:

```toml
[tool.poetry]
packages = [{include = "[name]"}]
```

... where `[name]` is the project name you pass at the `name` prompt. Basically, `poetry` assumes your project has a subdir that shares a name with your package.

This means `pip install` will fail unless:

* you either manually edit the package names in your `.toml`; or
* you move all top-level `.py` files in `project_a` into a subfolder named `project_a` again (or similar)

More on this in the last section below!

# Making `import` easy

Now, that we've got an easy, memorable way to install local packages, how do we make them easy to `import`  too? For example, look how nice this `import` flow is for Python's `random` library!

```python
import random
x = random.randrange(4)
```

However, if we try something similar with our package, we get an error instead!

```python
import project_a
x = project_a.the_module_i_wanna_import.a_func()
# AttributeError: module 'project_a' has no attribute 'the_module_i_wanna_import'
```

Note the language used in the error message here! When imported this way, Python treats `project_a` as its own module (rather than a parent package). With this kind of `import` statement, we only actually gain access to whatever's in the parent package's `__init__.py` (ie, nothing).

Python doesn't automatically give you immediate access to all the child modules like we might expect. In practice, it takes something more like the `import` below to get things right – and how ugly is that?

```python
import project_a.the_module_i_wanna_import
x = project_a.the_module_i_wanna_import.a_func()
```

To make `import` more intuitive, we can add an `import` line to the `__init__.py` of the package itself, saving our users (or future us) a lil bit of hassle.

```python
# __init__,py
import project_a.the_module_i_wanna_import as the_module_i_wanna_import
```

 With this added to our package-level `__init__.py`, our original `import` now works like a charm! Then just rinse and repeat, adding an `import` line to your `__init__.py` for each module in your package!

>Technically, you can leave out the `as the_module_i_wanna_import` clause, but there's a good reason for it! Without this, `project_a` ends up within its own scope - which means that `project_a.project_a` becomes a valid expression. If you rely on autocomplete for package/module names, this gets annoying fast. Stay up a little too late coding, and you'll start seeing arbitrarily nested `project_a.project_a.project_a.the_module_i_wanna_import` variants littered throughout your code!

# Don't like it? Have a cookie.

If you don't like the default values that `poetry` suggests (or the format of the generated `.toml`), that's "too bad" by `poetry` standards. In short, we don't have a config option for this.

Instead, like **[this StackOverflow thread](https://stackoverflow.com/questions/61428853/how-to-change-default-pyproject-toml-that-is-generated-when-running-poetry-ne)** suggests, you can use a project templating tool like `cookiecutter` to start new projects from template (which you can define/customize).

I gave `cookiecutter` a shot myself, and in short, **[here's what I came up with](https://github.com/ciraben/.cookie)**! I like what this tool offers because, as mentioned at the beginning of this article, I want a memorable, easy way to start new Python package projects, so that I can focus my inspired energy into those projects themselves.

Using `cookiecutter` is easy, even if designing your own "cookiecutter" (project template) takes a bit of getting used to. To use, simply navigate to the folder you normally add new projects to, and:

```zsh
$ cookiecutter https://github.com/ciraben/.cookie
```

Or you can `git clone` the repo into a `/cookie` directory within your projects folder. Then you can just run

```zsh
$ cookiecutter cookie
```

... and you'll be prompted through providing a new project name & details interactively!

___

_If you'd like learn more about creating your own custom `cookiecutter` template, check out **[the tutorial here](https://cookiecutter.readthedocs.io/en/2.0.2/tutorial2.html)**! Or if, you want to learn more about basic recommended project structures, check out **[the Python structure docs here](https://docs.python-guide.org/writing/structure/)** as well. (Keep in mind that most mention of `setup.py` can be replaced with `pyproject.toml` if you're using `poetry` instead of `setuptools`.)_
