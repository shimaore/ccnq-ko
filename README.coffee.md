Usage
=====

In the main application:

```coffeescript
ko = require 'knockout'
{Widget,widget} = (require 'ccnq-ko-widget') ko
html = ->
  widget 'foo'
ko.applyBindings foo: new Widget (...)
```

In the widget:

```coffeescript
module.exports = (require 'ccnq-ko').widget 'widget-tag', ->

  # data-model/constructor
  @data (value) -> @field = @ko.observable value
  # @constructor is a synonim for @data

  @view ({$root,data}) ->
  # any behavior
  # the fields of the constructor are already injected
  # the data object itself is available as {data}, {value} and {doc}
  @html -> {div} = @teacup # teacup+databind template as `this`
  @html ({div}) ->  # teacup+databind as parameter
```

Widget creation
===============

The name provided is expected to be the HTML tag, it should be dash-separated: `my-widget`

    widget = (tag_name,f) ->

The class name is: `MyWidget`

      re = /(^.|-.)/gi
      class_name = tag_name.replace re, (x) -> (x.substr x.length-1).toUpperCase()

The Teacup tag name: `my_widget`

      re = /-/g
      widget_name = tag_name.replace re, '_'

The function provided to `widget` is called with a small DSL:

      ctx =
        tag_name: tag_name
        class_name: class_name
        widget_name: widget_name

- `data` is the constructor for the data model; ideally it should work with `ko.toJS`

      ctx.data = ctx.constructor = (data) ->
        ctx._data = data

- `view` is the view model (behavior); the root of `this` is already populated with copies of the content of the object set by `data`

      ctx.view = (view) ->
        ctx._view = view

- `html` is the content as a Teacup template; teacup+databind is provided as parameter

      ctx.html = (html) ->
        ctx._html = html

      f.call ctx

The widget module returns a function that must be called with the instance of Knockout

      (ko) ->

        init ko

        ko.components.register tag_name,
          viewModel: ({value,$root}) ->
            for own k,v of value
              this[k] = v
            ctx._view.call this, {value,$root,ko,data:value,doc:value}
          template: teacup: ctx._html

        tag = (f) ->
          {tag} = teacup
          tag tag_name, params: "value:#{f},$root:$root"

        res = {tag_name,widget_name,class_name}

and returns a new Teacup widget

        res[widget_name] = tag

and the data-model constructor

        res[class_name] = ctx._data
        res

Loaders for Knockout
====================

    init = (ko) ->
      return if ko.__ccnq_init
      ko.__ccnq_init = true

      loader =

This allows us to dynamically render Teacup template. This solves for example problems with non-unique names for radio-buttons across multiple renderings.

        loadTemplate: (name, config, next) ->
          if config.teacup?
            ctx =
              teacup: teacup
            html = teacup.render config.teacup.call ctx, teacup
            ko.components.defaultLoader.loadTemplate name, html, next
          else
            next null

      ###
        loadViewModel: (name, config, next) ->
          if config.special
            constructor = ->
              config.special.apply this, arguments
            ko.components.defaultLoader.loadViewModel name, constructor, next
      ###

      ko.components.loaders.unshift loader

    teacup = require 'teacup'
    teacup.use (require 'teacup-databind')()

    module.exports = widget
    module.exports.widget = widget
