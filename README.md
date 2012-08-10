# carmen-rails Demo App

This app is a proof of concept to illustrate how to use
[carmen-rails](https://github.com/jim/carmen-rails) inside a simple form in a Rails
app.

carmen-rails depends upon [carmen](https://github.com/jim/carmen) for its
geographic data. If you want to customize the data generated by these helpers,
the Carmen docs explain how.

Throughout this example, it is assumed that the code is operating on an `Order`
model. Be sure to change all references to `order` to whatever is appropriate
for your application.

## Setup

Add `carmen-rails` to your Gemfile:

```ruby
gem 'carmen-rails'
```

Use the new form helpers provided by carmen-rails to add country or subregion
selects to your form:

```erb
<div class="field">
  <%= f.label :country_code %><br />
  <%= f.country_select :country_code, priority: %w(US CA), prompt: 'Please select a country' %>
</div>
```
 That's all there is to it. If you want to support a subregion field and have
 the options inside it updated by value in the country select, place this input
 inside a partial:

```erb
<div class="field">
  <%= f.label :state_code %><br />
  <%= render partial: 'subregion_select', locals: {parent_region: f.object.country_code} %>
</div>
```

Here is the content of the `subregion_select` partial:

```erb
<div id="order_state_code_wrapper">
  <% parent_region ||= params[:parent_region] %>
  <% country = Carmen::Country.coded(parent_region) %>

  <% if country.nil? %>
    <em>Please select a country above</em>
  <% elsif country.subregions? %>
    <%= subregion_select(:order, :state_code, parent_region) %>
  <% else %>
    <%= text_field(:order, :state_code) %>
  <% end %>
</div>
```

Note that we're defaulting to a text field input when we don't have subregion
information for a country, and that if we don't have a country at all, we show
a simple message.

Now we will add a small script that replaces the subregion select when the country
select's value changes. A wrapper div has been added to make this simpler.

```coffeescript
$ ->
  $('select#order_country_code').change (event) ->
    select_wrapper = $('#order_state_code_wrapper')

    $('select', select_wrapper).attr('disabled', true)

    country_code = $(this).val()

    url = "/orders/subregion_options?parent_region=#{country_code}"
    select_wrapper.load(url)
```

Now we just need to add a route to `config/routes.rb`:

```ruby
get '/orders/subregion_options' => 'orders#subregion_options'
```

And a basic controller action to the appropriate controller:

```ruby
def subregion_options
  render partial: 'subregion_select'
end
```

And that's basically it. This is obviously a very simple example, but it can
serve as the foundation of a variety of more involved interactions.


