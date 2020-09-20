---
layout: post
title:  "Blog-Hosting Odds & Ends"
date:   2020-09-19 16:00:00 -0700
categories: 
background: '/images/alley.jpg'
---

Last time, I wrote a bit about my process building this blog, from project start to `jekyll serve`. 
Here we'll go over a few more steps: pagination, Formspree, and Github Pages.

### Pagination

At some point, this blog will have more than two entries, and we'll need a way to archive them. 
A popular method is pagination, i.e. displaying entries over several numbered webpages - think Google search results.

Jekyll has a convenient plugin for this called `jekyll-paginate-v2`.

```bash
$ bundle add jekyll-paginate-v2
```

To use it, we need to add it to Jekyll's `_config.yml`:

```yaml
gems:
  - jekyll-paginate-v2
pagination:
  enabled: true
```

We can also customize how we want pages generated using [these](https://github.com/sverrirs/jekyll-paginate-v2/blob/master/README-GENERATOR.md) elements, like so:

```yaml
gems:
  - jekyll-paginate-v2
pagination:
  enabled: true
  per_page: 3
  permalink: '/posts/page/:num/'
  title: 'posts - page :num'
  limit: 0
  sort_field: 'date'
  sort_reverse: true
```

Then we need to enable it in the frontmatter of the page we want paginated.
For this blog, that's `posts/index.html`. 

```yaml
---
layout: page
pagination:
  enabled: true
---
```

To render our archive pages, we inject `pagination`'s [Liquid](https://github.com/Shopify/liquid) attributes into a post preview template. 
There are relevant instructions for it's predecessor on the [Jekyll site](https://jekyllrb.com/docs/pagination/) which I used as a reference.
Here's their Liquid template.

{% raw %}
```html
<!-- This loops through the paginated posts -->
{% for post in paginator.posts %}
  <h1><a href="{{ post.url }}">{{ post.title }}</a></h1>
  <p class="author">
    <span class="date">{{ post.date }}</span>
  </p>
  <div class="content">
    {{ post.content }}
  </div>
{% endfor %}

<!-- Pagination links -->
<div class="pagination">
  {% if paginator.previous_page %}
    <a href="{{ paginator.previous_page_path }}" class="previous">
      Previous
    </a>
  {% else %}
    <span class="previous">Previous</span>
  {% endif %}
  <span class="page_number ">
    Page: {{ paginator.page }} of {{ paginator.total_pages }}
  </span>
  {% if paginator.next_page %}
    <a href="{{ paginator.next_page_path }}" class="next">Next</a>
  {% else %}
    <span class="next ">Next</span>
  {% endif %}
</div>
```
{% endraw %}

The important thing to notice here is that `pagination` generates a `paginator` Liquid object  with all the useful info we need. 
Namely, `paginator.posts` lets us iterate over our posts to generate the archive.


Plug this into our `posts/index.html` template & we're good to go!
When we `jekyll serve`, Jekyll prints an additional output line to indicate successful pagination.

```bash
 Incremental build: disabled. Enable with --incremental
      Generating...
        Pagination: found page: posts/index.html
                            ...done in 1.320741 seconds.
```

We can see our results at [localhost:4000/posts](localhost:4000/posts).

### Formspree

The [theme](https://github.com/StartBootstrap/startbootstrap-clean-blog-jekyll) we're using for this blog comes packaged with the option of a contact page. In their setup guide, they offer this template as a starting point for a contact form:

```html
<form name="sentMessage" id="contactForm" novalidate>
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Name</label>
      <input type="text" class="form-control" placeholder="Name" id="name" required data-validation-required-message="Please enter your name.">
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Email Address</label>
      <input type="email" class="form-control" placeholder="Email Address" id="email" required data-validation-required-message="Please enter your email address.">
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <div class="control-group">
    <div class="form-group col-xs-12 floating-label-form-group controls">
      <label>Phone Number</label>
      <input type="tel" class="form-control" placeholder="Phone Number" id="phone" required data-validation-required-message="Please enter your phone number.">
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <div class="control-group">
    <div class="form-group floating-label-form-group controls">
      <label>Message</label>
      <textarea rows="5" class="form-control" placeholder="Message" id="message" required data-validation-required-message="Please enter a message."></textarea>
      <p class="help-block text-danger"></p>
    </div>
  </div>
  <br>
  <div id="success"></div>
  <div class="form-group">
    <button type="submit" class="btn btn-primary" id="sendMessageButton">Send</button>
  </div>
</form>
```

<img src="/images/contact1.png" class="w-100">

It looks nice but isn't functional out of the box. For that, we want Formspree.

[Formspree](http://formspree.io/) is a freemium web service that tracks form submissions & forwards them via email.
To use Formspree, first make an account. Then add your account's email address to your form html with 

```html
<form
  action="https://formspree.io/YOUR_EMAIL"
  method="POST"
>
```

Formspree offers this barebones contact form template (and [others](https://formspree.io/library)):

```html
<form
  action="https://formspree.io/FORM_ID"
  method="POST"
>
  <label>
    Your Name:
    <input type="text" name="name">
  </label>
  <label>
    Your Email:
    <input type="email" name="_replyto">
  </label>
  <label>
    Message:
    <textarea name="message"></textarea>
  </label>
  <input type="submit" value="Send">
</form>
```

<img src="/images/contact2.png" class="mw-100">

It looks pretty bland, but includes everything essential to function with Formspree.

Next, we need to validate the account. In order to do so, we need to submit our account email address into a working form. Let's test ours out with `jekyll serve`.

On sending the form submission, we are redirected to the Formspree site and an activation email is sent to us.
We can also check for successful submissions under the Forms tab on our Formspree account.

Once the form is activated, we can `git push` and set up our Github Pages site. 
Formspree identifies each form object by its host address, so once our form is up on Github, we'll have to activate it once again. 
(This also means we can edit the HTML now without worrying about reactivation.)

Now let's look back at our template's form template and see what we're missing.

First, we need to add 

```html
action="https://formspree.io/YOUR_EMAIL"
method="POST"
```

to the `<form>` element, as above. 

We'll also change that `<button>` element into an `<input>` while retaining those nice Bootstrap styles.

```html
<input type="submit" class="btn btn-primary" value="Send">
```

Next, we need to ensure the other `<input>` elements are each assigned a `name` value.
Formspree treats `name=_replyto` specially by passing the input string to the "Reply To" address in the email we receive - so let's add it to our "email" field.

Finally, I added some optional form fields to customize how submissions are processed.

```html
<input type="hidden" name="_subject" value="Sent from blog Contact form." />
<input type="text" name="_gotcha" style="display:none" />
<input type="hidden" name="_next" value="/" />
```

The first sets the subject line in emails we receive. The second is a fake input field to help weed out bots. And the third sets where users are redirected after form submission. 

Altogether, here's what I ended up with:

```html
  <div class="container">
    <div class="row">
      <div class="col-lg-8 col-md-10 mx-auto">

        <form id="contactform" action="//formspree.io/tom.on.github@gmail.com" method="POST" accept-charset="utf-8">

          <div class="control-group">
            <div class="form-group floating-label-form-group controls">
              <label>Name</label>
              <input type="text" name="name" class="form-control" placeholder="Name" required data-validation-required-message="Please enter your name.">
              <p class="help-block text-danger"></p>
            </div>
          </div>

          <div class="control-group">
            <div class="form-group floating-label-form-group controls">
              <label>Email Address</label>
              <input type="email" name="_replyto" class="form-control" placeholder="Email Address" required data-validation-required-message="Please enter your email address.">
              <p class="help-block text-danger"></p>
            </div>
          </div>

          <div class="control-group">
            <div class="form-group floating-label-form-group controls">
              <label>Message</label>
              <textarea name="message" class="form-control" placeholder="Message" required data-validation-required-message="Please enter a message."></textarea>
              <p class="help-block text-danger"></p>
            </div>
          </div>

          <input type="hidden" name="_subject" value="Sent from blog Contact form." />
          <input type="text" name="_gotcha" style="display:none" />
          <input type="hidden" name="_next" value="/" />

          <br>
          <div id="success"></div>
          <div class="form-group">
            <input type="submit" class="btn btn-primary" value="Send">
          </div>
        </form>
      </div>
    </div>
  </div>
```

### Using Github Pages with Jekyll 4

Github Pages is a great, free way to get your first website online. 
Furthermore, Github Pages is integrated with Jekyll out of the box, making webhosting even easier.
To set up a personal site:

1. Make a new repository on Github, called `<your_username>.github.io`.
2. After testing your site locally with `jekyll serve`, `git push` it to the new repository.
3. Go to your repository settings, scroll to the Github Pages section & confirm which branch to publish from.
4. In about 10 minutes, your website will be live at `https://<your_username>.github.io`.

Github takes care of building our site from the Jekyll components in our repository. 
However, it's only compatible with Jekyll 3.9.0. Meanwhile, Jekyll 4 has been released for over a year, and there's a bunch of cool new gems to try out.
How do we solve this problem?

The answer, until Github updates their Jekyll dependencies, is to use Github actions. 
(Another option is to host on [Netlify](https://www.netlify.com/blog/2020/04/02/a-step-by-step-guide-jekyll-4.0-on-netlify/) instead.)
Basically, actions are custom Github macros that can help with project deployment.

Because many people have been bumping up against this limit, Alain Hélaïli wrote [this action](https://github.com/helaili/jekyll-action) as a workaround.
`jekyll-action` assembles everything itself by running `jekyll build`, then pushes the output `_site` to a new branch for hosting.
Let's try it out!

First, add the subdirectory `.github/workflows/` to your project. 
This is where Github looks for actions. 
Next, following [these instructions](https://jekyllrb.com/docs/continuous-integration/github-actions/), create a [YAML](https://yaml.org/) file called `github-pages.yml` containing:

```yaml
name: Build and deploy Jekyll site to Github Pages

on:
  push:
    branches:
      - main

jobs:
  github-pages:
    runs-on: ubuntu-16.04
    steps:
      - uses: actions/checkout@v2
      - uses: helaili/jekyll-action@2.0.4
        env:
          JEKYLL_PAT: ${% raw %}{{ secrets.JEKYLL_PAT }}{% endraw %}
```

> Note: If you're working on another branch than `main`, substitute it for `main` above.
> See [the README](https://github.com/helaili/jekyll-action) for more options.

For `jekyll-action` to auto-push changes, it needs authentication. 
To set up an authentication token, follow these steps from the [Jekyll site](https://jekyllrb.com/docs/continuous-integration/github-actions/):

> 1.  On your GitHub profile, under Developer Settings, go to the Personal Access Tokens section.
> 2.  Create a token. Give it a name like “GitHub Actions” and ensure it has permissions to `public_repos` (or the entire `repo` scope for private repository) — necessary for the action to commit to the `gh-pages` branch.
> 3.  Copy the token value.
> 4.  Go to your repository’s Settings and then the Secrets tab.
> 5.  Create a token named `JEKYLL_PAT` (important). Give it a value using the value copied above.

Make sure to `add`, `commit` and `push` the new file as well.

Once `github-pages.yml` is up on Github, any further pushes will generate/update a site branch for us.
From there, all we have to do is select this branch in our repository settings to host Github Pages, and voilà - our Jekyll 4 site is up and running on Github.

Github actions are a powerful development tool. 
We really only grazed the surface here in order to get our site running. 
If you're curious for more, check out Sid's take on actions [here](https://devopsdirective.com/posts/2020/07/stupid-github-actions/).
