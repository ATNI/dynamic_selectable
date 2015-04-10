# DynamicSelectable

**DynamicSelectable** is a [Rails](http://github.com/rails/rails) gem that allows you to easily create `collection_select` fields with results that dynamically populate a related field. The dynamic population occurs with the help of a little piece of [CoffeeScript](http://coffeescript.org/) called [*jquery-dynamic-selectable*](http://railsguides.net/cascading-selects-with-ajax-in-rails/) written by [Andrey Koleshko](http://railsguides.net/about-author/).

How about a use case? Let's say your application allowed users to look up parts for their vehicle. They would probably select their vehicle as follows:

1. User selects the `Year` of their vehicle
2. The `Make` field is populated with makes available in the selected `Year`
3. User selects the `Make` of their vehicle
4. The `Model` field is populated with models of the selected `Make`
5. User selects the `Model` of their vehicle
6. The `Engine` field is populated with engine types for the selected `Model`
7. User selects the `Engine` of their vehicle

In order to facilititate this with *jquery-dynamic-selectable*, you need to create a controller to call with your parent `collection_select`'s value which brings back [JSON](http://json.org/) used to populate the related field. This also requires some routing, and a few lengthy additions to the `html_options`. However, as you'll see in the usage below, **DynamicSelectable** will allow you to easily generate the necessary controller & route, and it gives you a nice `FormHelper` method which saves you the trouble of setting lengthy `html_options`.

## Installation

Add these lines to your application's Gemfile:

```ruby
gem 'coffee-script'
gem 'dynamic_selectable', git: 'https://github.com/atni/dynamic_selectable.git'
```

Install the gems by running `bundle install`.

Next we install the **DynamicSelectable** gem to set up our routes and parent controller:

```bash
$ rails generate dynamic_selectable:install
      insert  config/routes.rb
      create  app/controllers/select/select_controller.rb
```

If you are using Devise for user authentication, you will want to uncomment the line in the newly generated `app/controllers/select/select_controller.rb`.

```ruby
class Select::SelectController < ApplicationController
  skip_before_filter :authenticate_user!
end
```

If you are using a model other than `User` for [Devise](https://github.com/plataformatec/devise) you will need to specify it properly here. You can modify this parent controller to make any global changes to other Select controllers generated by this gem.

Now that you have **DynamicSelectable** installed, you're ready to start creating some `dynamic_collection_select` fields in your application!

## Usage

The first step to creating a `dynamic_collection_select` is to generate a controller and route for it. Using the vehicle example from above, your `Make` and `Model` models might look like the following:

```ruby
# == Schema Information
#
# Table name: makes
#
#  id         :integer          not null, primary key
#  name       :string
#  created_at :datetime         not null
#  updated_at :datetime         not null
#

class Make < ActiveRecord::Base
end
```

```ruby
# == Schema Information
#
# Table name: models
#
#  id         :integer          not null, primary key
#  name       :string
#  make_id    :integer
#  created_at :datetime         not null
#  updated_at :datetime         not null
#

class Model < ActiveRecord::Base
  belongs_to :make
end
```

To generate the controller and route necessary for your selection of `Make` to dynamically populate your selection of `Model`, you would run the following:

```bash
$ rails generate dynamic_selectable:select make model id name name:asc
```

Your routes will now contain the following:

```ruby
  namespace :select do
    get ':make_id/models', to: 'models#index', as: :models
  end
```

and the following controller will be generated:

```ruby
class Select::ModelsController < Select::SelectController
  def index
    models = Model.where(make_id: params[:make_id]).select('id, name').order('name asc')
    render json: models
  end
end
```

Now all that is left is to build your form. The method `dynamic_collection_select` is defined as:

<dl>
  <dt>current</dt>
  <dd>Symbol representing the class of the object being built in the current form</dd>

  <dt>select_parent</dt>
  <dd>The parent field that will determine the population of the related child field</dd>

  <dt>select_child</dt>
  <dd>The child field that will be populated based on the selection in the parent field</dd>

  <dt>collection</dt>
  <dd>Collection to populate the parent field with</dd>

  <dt>value_method</dt>
  <dd>Method to obtain the value of objects in the parent collection</dd>

  <dt>text_method</dt>
  <dd>Method to obtain the text of objects in the parent collection</dd>

  <dt>options</dt>
  <dd>Provide options like :include_blank</dd>

  <dt>html_options</dt>
  <dd>Provide options interpreted as HTML such as styling</dd>
</dl>

```ruby
def dynamic_collection_select(current, select_parent, select_child, collection,
                              value_method, text_method, options, html_options)
```

A very simple form would look something like this:

```html+erb
<%= form_for(@vehicle) do |f| %>
  <div class="form-group">
    <%= label_tag :make_id %>
    <%= dynamic_collection_select :vehicle, :make, :model, Make.all, :id, :name,
        { include_blank: true }, {} %>
  </div>

  <div class="form-group">
    <%= f.label :model_id %>
    <%= f.collection_select :serving_area_id, [], :id, :name, {}, { class: 'form-control' } %>
  </div>

  <%# ... %>
<% end %>
```

That's it. Happy selecting!

## Contributing

1. Fork it ( https://github.com/atni/dynamic_selectable/fork )
2. Create your feature branch (`git checkout -b my-new-feature`)
3. Commit your changes (`git commit -am 'Add some feature'`)
4. Push to the branch (`git push origin my-new-feature`)
5. Create a new Pull Request