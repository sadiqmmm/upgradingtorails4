## <a id="asset-pipeline"></a>Asset Pipeline

Managing assets like JavaScript, CSS stylesheets, and images was made much
easier with the introduction of the asset pipeline in Rails 3.1.

From the perspective of your Rails application, there are not many changes to
the asset pipeline in Rails 4. Internally, on the other hand, there are many
changes. For example, performance has been dramatically improved: you should
notice that asset precompilation during a deploy takes much less time in Rails
4 than in Rails 3.

That said, there are some gotchas you need to be aware of when upgrading.

### <a id="js-compress"></a>JavaScript and CSS Compressors

Rails 4 changes the configuration options surrounding compressing assets.

By default, Rails 3 enabled JavaScript and CSS compression with the
`config.assets.compress` directive in `config/assets/production.rb`:

@@@ ruby
# config/environments/production.rb
Widgets::Application.configure do
  # Rails 3 setting
  config.assets.compress = true
end
@@@

Instead, Rails 4 requires you to explicitly specify which compressors to use.
In fact, the `config.assets.compress` setting silently has no effect any
longer. If you do not update your configuration, assets will no longer be
compressed, which could lead to sluggish page loads in production.

When upgrading, remove the `config.assets.compress` directive and replace it
with `js_compressor` and `css_compressor`:

@@@ ruby
# config/environments/production.rb
Widgets::Application.configure do
  # Remove config.assets.compress = true
  config.assets.js_compressor  = :uglifier
  config.assets.css_compressor = :sass
end
@@@

While other compressors can be used, `uglifier` and `sass` are available by
default in both Rails 3 and 4.

### <a id="precompiled-images"></a>Precompiled Assets in lib/ and vendor/

Assets (JavaScript, CSS stylesheets, and images) can be added to three
directories in a Rails application: `app/assets`, `lib/assets` and
`vendor/assets`. Application-specific assets are generally placed in
`app/assets`, and assets for third-party libraries are generally placed in
`lib/assets` or `vendor/assets`.

Rails 4 no longer automatically precompiles image assets residing in `lib/` and
`vendor/` during a deploy. JavaScript and CSS stylesheets in those directories
likely *not* affected by this change.

If your application has image assets in `lib/` or `vendor/`, you must either:

1. Move the images to `app/assets/images` (removing them from
   `lib/assets/images` or `vendor/assets/images`)
1. Add the image filenames to the `config.assets.precompile` list in
   `config/environments/production.rb` or `config/application.rb`

Consider an image named `sprites.png` that resides in `vendor/assets/images`.
In Rails 3, this image would be automatically precompiled. In Rails 4, however,
it needs to be added to the list of assets in `config.assets.precompile`:

@@@ ruby
# config/environments/production.rb
Widgets::Application.configure do
  # ...

  config.assets.precompile = %w( sprites.png )
end
@@@

Frustratingly, `sprites.png` will show up correctly in development regardless
of whether it has been added to the precompile list. If it is not added to
the list, though, it will break when you deploy to production.

<!-- TODO: Talk about gems who may need to move assets to app? -->
