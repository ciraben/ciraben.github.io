---
layout: page
title: contact
description: 
background: '/images/snowdrops.jpg'
---

<p>
Questions? Comments? Limited-time offers?
<br>
<br>
Forget your email client, just shoot me a quick one right here!
</p>

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
    <textarea name="message" class="form-control" placeholder="Message" required data-validation-required-message="Please enter a message." rows=5></textarea>
      <p class="help-block text-danger"></p>
    </div>
  </div>

    <!--the following are optional fields to customize how submissions are processed-->
    <!--The first sets email subject.-->
    <!--The second catches spambots.-->

  <input type="hidden" name="_subject" value="Sent from blog Contact form." />
  <input type="text" name="_gotcha" style="display:none" />
  <input type="hidden" name="_next" value="#" />

  <br>
  <div id="success"></div>
  <div class="form-group">
    <input type="submit" class="btn btn-primary" value="Send">
  </div>
</form>

      </div>
    </div>
  </div>
