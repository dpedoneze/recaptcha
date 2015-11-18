# reCAPTCHA

Author:    Jason L Perry (http://ambethia.com)<br/>
Copyright: Copyright (c) 2007-2013 Jason L Perry<br/>
License:   [MIT](http://creativecommons.org/licenses/MIT/)<br/>
Info:      https://github.com/ambethia/recaptcha<br/>
Bugs:      https://github.com/ambethia/recaptcha/issues<br/>

This plugin adds helpers for the [reCAPTCHA API](https://www.google.com/recaptcha). In your
views you can use the `recaptcha_tags` method to embed the needed javascript,
and you can validate in your controllers with `verify_recaptcha` or `verify_recaptcha!`,
which throws an error on failiure.

Beforehand you need to configure Recaptcha with your custom private and public
key. You may find detailed examples below. Exceptions will be raised if you
call these methods and the keys can't be found.

## Rails Installation

```Ruby
gem "recaptcha", require: "recaptcha/rails"
```

(Rails < 3.0: install an older release and view it's README)

## Setting up your API Keys

There are multiple ways to setup your reCAPTCHA API key once you
[obtain a pair](https://www.google.com/recaptcha/admin).

### Recaptcha.configure

You may use the block style configuration. The following code could be placed
into a `config/initializers/recaptcha.rb` when used in a Rails project.

```Ruby
Recaptcha.configure do |config|
  config.public_key  = '6Lc6BAAAAAAAAChqRbQZcn_yyyyyyyyyyyyyyyyy'
  config.private_key = '6Lc6BAAAAAAAAKN3DRm6VA_xxxxxxxxxxxxxxxxx'
  # Uncomment the following line if you are using a proxy server:
  # config.proxy = 'http://myproxy.com.au:8080'
  # Uncomment if you want to use the older version of the API,
  # only works for versions >= 0.3.7, default value is 'v2':
  # config.api_version = 'v1'
end
```

This way, you may also set additional options to fit recaptcha into your
deployment environment.

### Recaptcha.with_configuration

If you want to temporarily overwrite the configuration you set with 
`Recaptcha.configure` (when testing, for example), you can use a 
`Recaptcha#with_configuration` block:

```Ruby
Recaptcha.with_configuration(public_key: '12345') do
  # Do stuff with the overwritten public_key.
end
```

### Heroku & Shell environment

Or, you can keep your keys out of your code base by exporting the following
environment variables. You might do this in the .profile/rc, or equivalent for
the user running your application. This would also be the preffered method
in an Heroku deployment.

```
export RECAPTCHA_PUBLIC_KEY  = '6Lc6BAAAAAAAAChqRbQZcn_yyyyyyyyyyyyyyyyy'
export RECAPTCHA_PRIVATE_KEY = '6Lc6BAAAAAAAAKN3DRm6VA_xxxxxxxxxxxxxxxxx'
```

### Per call

You can also pass in your keys as options at runtime, for example:

```Ruby
recaptcha_tags public_key: '6Lc6BAAAAAAAAChqRbQZcn_yyyyyyyyyyyyyyyyy'
```

and later,

```Ruby
verify_recaptcha private_key: '6Lc6BAAAAAAAAKN3DRm6VA_xxxxxxxxxxxxxxxxx'
```

This option might be useful, if the same code base is used for multiple
reCAPTCHA setups.

## To use 'recaptcha'

Add `recaptcha_tags` to each form you want to protect. 
Place it where you want the recaptcha widget to appear.

Example:

```Erb
<%= form_for @foo do |f| %>
  # ... additional lines truncated for brevity ...
  <%= recaptcha_tags %>
  # ... additional lines truncated for brevity ...
<% end %>
```

And, add `verify_recaptcha` logic to each form action that you've protected.

### recaptcha_tags

Some of the options available:

| Option      | Description |
|-------------|-------------|
| :ssl        | Uses secure http for captcha widget (default `false`, but can be changed by setting `config.use_ssl_by_default`)|
| :noscript   | Include <noscript> content (default `true`)|
| :display    | Takes a hash containing the `theme` and `tabindex` options per the API. (default `nil`), options: 'red', 'white', 'blackglass', 'clean', 'custom'|
| :ajax       | Render the dynamic AJAX captcha per the API. (default `false`)|
| :public_key | Your public API key, takes precedence over the ENV variable (default `nil`)|
| :error      | Override the error code returned from the reCAPTCHA API (default `nil`)|

You can also override the html attributes for the sizes of the generated `textarea` and `iframe`
elements, if CSS isn't your thing. Inspect the source of `recaptcha_tags` to see these options.

### verify_recaptcha

This method returns `true` or `false` after processing the parameters from the reCAPTCHA widget. Why
isn't this a model validation? Because that violates MVC. You can use it like this, or how ever you
like. Passing in the ActiveRecord object is optional, if you do--and the captcha fails to verify--an
error will be added to the object for you to use.

Some of the options available:

| Option       | Description |
|--------------|-------------|
| :model       | Model to set errors
| :attribute   | Model attribute to receive errors (default :base)
| :message     | Custom error message
| :private_key | Your private API key, takes precedence over the ENV variable (default `nil`).
| :timeout     | The number of seconds to wait for reCAPTCHA servers before give up. (default `3`)

```Ruby
respond_to do |format|
  if verify_recaptcha(model @post, message: "Oh! It's error with reCAPTCHA!") && @post.save
    # ...
  else
    # ...
  end
end
```

### Add multiple widgets to the same page

This is an example taken from the [official google documentation](https://developers.google.com/recaptcha/docs/display).

Add a script tag for a callback

```Html
<script type="text/javascript">
  var verifyCallback = function(response) {
    alert(response);
  };
  var widgetId1;
  var widgetId2;
  var onloadCallback = function() {
    // Renders the HTML element with id 'example1' as a reCAPTCHA widget.
    // The id of the reCAPTCHA widget is assigned to 'widgetId1'.
    widgetId1 = grecaptcha.render('example1', {
      'sitekey' : "<%= Recaptcha.configuration.public_key %>",
      'theme' : 'light'
    });
    widgetId2 = grecaptcha.render(document.getElementById('example2'), {
      'sitekey' : "<%= Recaptcha.configuration.public_key %>"
    });
    grecaptcha.render('example3', {
      'sitekey' : "<%= Recaptcha.configuration.public_key %>",
      'callback' : verifyCallback,
      'theme' : 'dark'
    });
  };
</script>
```

In the callback you will have the `sitekey` generated by this gem with `<%= Recaptcha.configuration.public_key %>`

Next you need to have some elements with an id matching those specified in the callback

```Erb
<%= form_for @foo do |f| %>
  # ... additional lines truncated for brevity ...
  <div id="example1"></div>
  # ... additional lines truncated for brevity ...
<% end %>
<%= form_for @foo2 do |f| %>
  # ... additional lines truncated for brevity ...
  <div id="example2"></div>
  # ... additional lines truncated for brevity ...
<% end %>
<%= form_for @foo3 do |f| %>
  # ... additional lines truncated for brevity ...
  <div id="example3"></div>
  # ... additional lines truncated for brevity ...
<% end %>
```

And finally you need a script tag that gets the reCAPTCHA code from google and tells it that your code is explicit and gives it the callback.

```Html
<script src="https://www.google.com/recaptcha/api.js?onload=onloadCallback&render=explicit" async defer></script>
```

Now all together:

```Html
<html>
  <head>
    <title>reCAPTCHA demo: Explicit render for multiple widgets</title>
    <script type="text/javascript">
      var verifyCallback = function(response) {
        alert(response);
      };
      var widgetId1;
      var widgetId2;
      var onloadCallback = function() {
        // Renders the HTML element with id 'example1' as a reCAPTCHA widget.
        // The id of the reCAPTCHA widget is assigned to 'widgetId1'.
        widgetId1 = grecaptcha.render('example1', {
          'sitekey' : "<%= Recaptcha.configuration.public_key %>",
          'theme' : 'light'
        });
        widgetId2 = grecaptcha.render(document.getElementById('example2'), {
          'sitekey' : "<%= Recaptcha.configuration.public_key %>"
        });
        grecaptcha.render('example3', {
          'sitekey' : "<%= Recaptcha.configuration.public_key %>",
          'callback' : verifyCallback,
          'theme' : 'dark'
        });
      };
    </script>
  </head>
  <body>
    <%= form_for @foo do |f| %>
      # ... additional lines truncated for brevity ...
      <div id="example1"></div>
      # ... additional lines truncated for brevity ...
    <% end %>
    <%= form_for @foo2 do |f| %>
      # ... additional lines truncated for brevity ...
      <div id="example2"></div>
      # ... additional lines truncated for brevity ...
    <% end %>
    <%= form_for @foo3 do |f| %>
      # ... additional lines truncated for brevity ...
      <div id="example3"></div>
      # ... additional lines truncated for brevity ...
    <% end %>
    <script src="https://www.google.com/recaptcha/api.js?onload=onloadCallback&render=explicit" async defer></script>
  </body>
</html>
```

The only real difference between this example and the google example is you will use the `<%= Recaptcha.configuration.public_key %>` for the `sitekey`

If your callback has to live on another file (maybe a layout), then you would set the callback on window `window.onloadCallback = function() {...}`

Then on the backend, you will still use the `verify_recaptcha` as explained in this readme.

## I18n support
reCAPTCHA passes two types of error explanation to a linked model. It will use the I18n gem
to translate the default error message if I18n is available. To customize the messages to your locale,
add these keys to your I18n backend:

`recaptcha.errors.verification_failed` error message displayed if the captcha words didn't match
`recaptcha.errors.recaptcha_unreachable` displayed if a timeout error occured while attempting to verify the captcha

Also you can translate API response errors to human friendly by adding translations to the locale (`config/locales/en.yml`):

```Yaml
en:
  recaptcha:
    errors:
      incorrect-captcha-sol: 'Fail'
```

## Testing
By default, reCAPTCHA is skipped on "test" env, so if you need to make sure it's working properly, just remove the "test" entry of the skip_environment inside Recapcha class, look:


```Ruby
Recaptcha.configuration.skip_verify_env.delete("test")
```
