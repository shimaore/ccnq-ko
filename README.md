KnockOut(JS) Widget
===================

An opinionated widget constructor for Browserify and CoffeeScript, using [KnockOutJS](http://knockoutjs.com/) and [teacup](http://goodeggs.github.io/teacup/)+[databind](https://github.com/shimaore/teacup-databind).

Usage
=====

In the main application:

```coffeescript
ko = require 'knockout'
{MyWidget,my_widget} = (require 'kow-my-widget') ko

# Later...
teacup = require 'teacup'
$('content').html = teacup.render ->
  my_widget 'foo'
# Provide default values for the fields.
values = new MyWidget 'bar'
ko.applyBindings foo: values

# Later...
field_value = values.observable_field()
```

In `kow-my-widget`:

```coffeescript
module.exports = (require 'kow') 'my-widget', ->

  ## The parameter of @data becomes MyWidget.
  ## @constructor and @model are synonims for @data
  ## You might also provide a CoffeeScript class instead of a function.
  @data (value) ->
    @observable_field = @ko.observable value

  @view ({$root,data}) ->
  ## the fields of the constructor are already injected in the data object
  ## the data object is available as {data}, {value} and {doc}

  @html ({div,input}) ->   # teacup+databind as parameter
    div '.layout', ->
      input '.name', ->
        bind:
          value: 'observable_field'
```
