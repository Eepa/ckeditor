# Ckeditor

CKEditor is a ready-for-use HTML text editor designed to simplify web content creation. It's a WYSIWYG editor that brings common word processor features directly to your web pages. Enhance your website experience with our community maintained editor.
[ckeditor.com](http://ckeditor.com/)

## Features

* Downgraded gem version to 4.1.1 in order to avoid cktext_area load error (a bit [like](https://github.com/galetahub/ckeditor/issues/572))
* Rails rake assets:precompile [issue](https://github.com/galetahub/ckeditor/issues/551) fixed like proposed [by mkaszubowski](https://github.com/mkaszubowski/ckeditor/commits/master)
* Ckeditor version 4.4.6 (full)
* Rails 4 integration
* Files browser
* HTML5 files uploader
* Hooks for formtastic and simple_form forms generators
* Integrated with authorization framework CanCan and Pundit

## Installation

For basic usage just include ckeditor gem:

```
gem 'ckeditor'
```

or if you want to use the latest version from github:

```
gem 'ckeditor', github: 'galetahub/ckeditor'
```

#### Using with ruby 1.8.7

For usage with ruby 1.8.7 you need to specify gem version:

```
gem 'ckeditor', '4.0.4'
```

For files uploading support you need generate models for file storage.
Currently supported next backends:

* ActiveRecord (paperclip, carrierwave, dragonfly)
* Mongoid (paperclip, carrierwave, dragonfly)

### How generate models for store uploading files

#### ActiveRecord + paperclip

For active_record orm is used paperclip gem (it's by default).

```
gem 'paperclip'

rails generate ckeditor:install --orm=active_record --backend=paperclip
```

#### ActiveRecord + carrierwave

```
gem 'carrierwave'
gem 'mini_magick'

rails generate ckeditor:install --orm=active_record --backend=carrierwave
```

#### Mongoid + paperclip

```
gem 'mongoid-paperclip', :require => 'mongoid_paperclip'

rails generate ckeditor:install --orm=mongoid --backend=paperclip
```

#### Mongoid + carrierwave

```
gem 'carrierwave-mongoid', :require => 'carrierwave/mongoid'
gem 'mini_magick'

rails generate ckeditor:install --orm=mongoid --backend=carrierwave
```

#### Load generated models

All ckeditor models will be generated into app/models/ckeditor folder.
Models are autoloaded in Rails 4. For earlier Rails versions you need to add them to autoload path (in application.rb):

```ruby
config.autoload_paths += %W(#{config.root}/app/models/ckeditor)
```

Mount engine in your routes (config/routes.rb):

```ruby
mount Ckeditor::Engine => '/ckeditor'
```

## Usage

Include ckeditor javascripts in your `app/assets/javascripts/application.js`:

```
//= require ckeditor/init
```

Form helpers:

```erb
<%= form_for @page do |form| -%>
  ...
  <%= form.cktext_area :notes, :class => 'someclass', :ckeditor => {:language => 'uk'} %>
  ...
  <%= form.cktext_area :content, :value => 'Default value', :id => 'sometext' %>
  ...
  <%= cktext_area :page, :info, :cols => 40, :ckeditor => {:uiColor => '#AADC6E', :toolbar => 'mini'} %>
  ...
<% end -%>
```

### Customize ckeditor

All ckeditor options [here](http://docs.ckeditor.com/#!/api/CKEDITOR.config)

In order to configure the ckeditor default options, create files:

```
app/assets/javascripts/ckeditor/config.js

app/assets/javascripts/ckeditor/contents.css
```

#### Custom toolbars example

Adding custom toolbar:

```javascript
# in app/assets/javascripts/ckeditor/config.js

CKEDITOR.editorConfig = function (config) {
  // ... other configuration ...
  
  config.toolbar_mini = [
    ["Bold",  "Italic",  "Underline",  "Strike",  "-",  "Subscript",  "Superscript"],
  ];
  config.toolbar = "simple";

  // ... rest of the original config.js  ...
}
```

When overriding default `config.js` you have to set all configuration yourself since bundled `config.js` will not be loaded. To see the default configuration run `bundle open ckeditor` and copy `app/assets/javascripts/ckeditor/config.js` to your project and customize it to your needs.

### Deployment

For Rails 4, add the following to `config/initializers/assets.rb`:

```ruby
Rails.application.config.assets.precompile += %w( ckeditor/* )
```

Since version 4.1.0, non-digested assets of ckeditor will simply be copied after digested assets were compiled.
For older versions, use gem [non-stupid-digest-assets](https://rubygems.org/gems/non-stupid-digest-assets), to copy non digest assets.

To reduce the asset precompilation time, you can limit plugins and/or languages to those you need:

```ruby
# in config/initializers/ckeditor.rb

Ckeditor.setup do |config|
  config.assets_languages = ['en', 'fr']
  config.assets_plugins = ['image', 'smiley']
end
```

Note that you have to list your plugins including all their dependencies.

### Include customized CKEDITOR_BASEPATH setting

Add your app/assets/javascripts/ckeditor/basepath.js.erb like

```erb
<%
  base_path = ''
  if ENV['PROJECT'] =~ /editor/i
    base_path << "/#{Rails.root.basename.to_s}/"
  end
  base_path << Rails.application.config.assets.prefix
  base_path << '/ckeditor/'
%>
var CKEDITOR_BASEPATH = '<%= base_path %>';
```

### AJAX

jQuery sample:

```html
<script type='text/javascript' charset='UTF-8'>
  $(document).ready(function(){
    $('form[data-remote]').bind('ajax:before', function(){
      for (instance in CKEDITOR.instances){
        CKEDITOR.instances[instance].updateElement();
      }
    });
  });
</script>
```

### Formtastic integration

```erb
<%= form.input :content, :as => :ckeditor %>
<%= form.input :content, :as => :ckeditor, :input_html => { :ckeditor => { :height => 400 } } %>
```

### SimpleForm integration

```erb
<%= form.input :content, :as => :ckeditor, :input_html => { :ckeditor => {:toolbar => 'Full'} } %>
```

### CanCan integration

To use cancan with Ckeditor, add this to an initializer.

```ruby
# in config/initializers/ckeditor.rb

Ckeditor.setup do |config|
  config.authorize_with :cancan
end
```

At this point, all authorization will fail and no one will be able to access the filebrowser pages.
To grant access, add this to Ability#initialize

```ruby
# Always performed
can :access, :ckeditor   # needed to access Ckeditor filebrowser

# Performed checks for actions:
can [:read, :create, :destroy], Ckeditor::Picture
can [:read, :create, :destroy], Ckeditor::AttachmentFile
```

### Pundit integration

Just like CanCan, you can write this code in your config/initializers/ckeditor.rb file

```ruby
Ckeditor.setup do |config|
  config.authorize_with :pundit
end
```

And then, generate the policy files for model **Picture** and **AttachmentFile**

```
$ rails g ckeditor:pundit_policy
```
By this command, you will got two files:
> app/policies/ckeditor/picture_policy.rb
app/policies/ckeditor/attachment_file_policy.rb

By default, only the user that logged in can access the models(with action *index* and *create*), and only the owner of the asset can **destroy** the resource.

You can simply customize these two policy files as you like.

## I18n

```yml
en:
  ckeditor:
    page_title: 'CKEditor Files Manager'
    confirm_delete: 'Delete file?'
    buttons:
      cancel: 'Cancel'
      upload: 'Upload'
      delete: 'Delete'
      next: 'Next'
```

## Tests

```bash
$> rake test
$> rake test CKEDITOR_ORM=mongoid
$> rake test CKEDITOR_BACKEND=carrierwave

$> rake test:controllers
$> rake test:generators
$> rake test:integration
$> rake test:models
```

This project rocks and uses MIT-LICENSE.
