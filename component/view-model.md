@property {Object|can.Map|function} can.Component.prototype.viewModel
@parent can.Component.prototype

@description

Provides or describes a [can.Map] constructor function or `can.Map` instance that will be used to retrieve values found in the component's [can.Component::template template]. The map instance is initialized with values specified by attributes set on the component's HTML element.

__Note:__ In 2.1, [can.stache] and [can.mustache] pass values to the
viewModel differently. To pass data from the viewModel, you must wrap your attribute 
value with `{}`. In 3.0, `can.mustache`
will use `can.stache`'s syntax.



@option {Object} A plain JavaScript object that is used to define the prototype methods and properties of the [can.Construct constructor function] that extends [can.Map]. For example:

    can.Component.extend({
      tag: "my-paginate",
      viewModel: {
        offset: 0,
        limit: 20,
        next: function(){
          this.attr("offset", this.offset + this.limit);
        }
      }
    })

Prototype properties that have values of `"@"` are not looked up in the current viewModel, instead the literal string value of the relevant attribute is used (like pass by value instead of pass by reference).  For example:

    can.Component.extend({
      tag: "my-tag",
      viewModel: {
        title: "@"
      },
      template: "<h1>{{title}}</h1>"
    });
   
With source HTML like:

    <my-tag title="hello"></my-tag>
    
Results in:

    <my-tag><h1>hello</h1></my-tag>

@option {can.Map} A `can.Map` constructor function will be used to create an instance of the observable `can.Map` used as the template's viewModel.  For example:

    var Paginate = can.Map.extend({
      offset: 0,
      limit: 20,
      next: function(){
        this.attr("offset", this.offset + this.limit);
      }
    })
    can.Component.extend({
      tag: "my-paginate",
      viewModel: Paginate
    })

@option {function} Returns the instance or constructor function of the object that will be added to the viewModel.

@param {Object} attrs An object of values specified by the custom element's attributes. For example, a template rendered like:

    can.stache("<my-element title="name"></my-element>")({
      name: "Justin"
    })

Creates an instance of the following component:

    can.Component.extend({
    	tag: "my-element",
    	viewModel: function(attrs){
    	  attrs.title //-> "Justin";
    	  return new can.Map(attrs);
    	}
    })

And calls the viewModel function with `attrs` like `{title: "Justin"}`.

@param {can.view.viewModel} parentviewModel

The viewModel the custom tag was found within.  By default, any attribute's values will be looked up within the current viewModel, but if you want to add values without needing the user to provide an attribute, you can set this up here.  For example:

    can.Component.extend({
    	tag: "my-element",
    	viewModel: function(attrs, parentViewModel){
    	  return new can.Map({title: parentViewModel.attr('name')});
    	}
    });

Notice how the attribute's value is looked up in `my-element`'s parentViewModel.

@param {HTMLElement} element The element the [can.Component] is going to be placed on. If you want to add custom attribute handling, you can do that here.  For example:

    can.Component.extend({
    	tag: "my-element",
    	viewModel: function(attrs, parentViewModel, el){
    	  return new can.Map({title: el.getAttribute('title')});
    	}
    });

@return {can.Map|Object} Specifies one of the following:

 - The data used to render the component's template.
 - The prototype of a `can.Map` that will be used to render the component's template.
 
@option {can.Map} If an instance of `can.Map` is returned, that instance is placed
on top of the viewModel and used to render the component's template.

@option {Object} If a plain JavaScript object is returned, that is used as a prototype
definition used to extend `can.Map`.  A new instance of the extended Map is created.

@body

## Use

[can.Component]'s viewModel property is used to define an __object__, typically an instance
of a [can.Map], that will be used to render the component's 
template. This is most easily understood with an example.  The following
component shows the current page number based off a `limit` and `offset` value:

    can.Component.extend({
      tag: "my-paginate",
      viewModel: {
        offset: 0,
        limit: 20,
        page: function(){
          return Math.floor(this.attr('offset') / this.attr('limit')) + 1;
        }
      },
      template: "Page {{page}}."
    })

If this component HTML was inserted into the page like:

    var template = can.stache("<my-paginate></my-paginate>")
    $("body").append(template())

It would result in:

    <my-paginate>Page 1</my-paginate>
    
This is because the provided viewModel object is used to extend a can.Map like:

    CustomMap = can.Map.extend({
      offset: 0,
      limit: 20,
      page: function(){
        return Math.floor(this.attr('offset') / this.attr('limit')) + 1;
      }
    })

Any primitives found on a `can.Map`'s prototype (ex: `offset: 0`) are used as
default values.

Next, a new instance of CustomMap is created with the attribute data within `<my-paginate>`
(in this case there is none) like:

    componentData = new CustomMap(attrs);
    
And finally, that data is added to the `parentviewModel` of the component, used to 
render the component's template, and inserted into the element:

    var newviewModel = parentviewModel.add(componentData),
        result = can.stache("Page {{page}}.")(newviewModel);
    $(element).html(result);

## Values passed from attributes

Values can be "passed" into the viewModel of a component, similar to passing arguments into a function. By default, 
custom tag attributes (other than `class` and `id`) are looked up
in the parent viewModel and set as observable values on the [can.Map] instance. 

Values passed in this way are passed similar to function arguments that are "pass by reference" because they are crossbound to 
the property in the parent viewModel. Changes in the parent viewModel property that is passed in trigger changes in this component's viewModel 
property, and likewise, changes in the component's viewModel property trigger a change in the parent viewModel property.

As mentioned in the deprecation warning above, using [can.stache], values are passed into components like this:

    <my-paginate offset='{index}' limit='{size}'></my-paginate>

But using [can.mustache], values are passed like this:

    <my-paginate offset='index' limit='size'></my-paginate>

The rest of the examples in this section show `can.stache` syntax.

The next example would create an offset and limit property on the component that are one-way bound (from parent to child) to the values of index and size.

    <my-paginate offset='{{index}}' limit='{{size}}'></my-paginate>

The one-way binding is created when using double brackets.  It would NOT two-way bind the `offset` and `limit` properties to the `index` and `size`. So if the `<my-paginate>` component changed the value of `offset` in its viewModel, the value of `index` would not update.

The following component requires an `offset` and `limit` to be passed in:

    can.Component.extend({
      tag: "my-paginate",
      viewModel: {
        page: function(){
          return Math.floor(this.attr('offset') / this.attr('limit')) + 1;
        },
        offset: null,
        limit: null
      },
      template: "Page {{page}}."
    });

If `<my-paginate>`'s source html is rendered like...

    var template = can.stache("<my-paginate offset='{index}' limit='{size}'></my-paginate>");
    
    var pageInfo = new can.Map({
      index: 0,
      size: 20
    });
    
    $("body").append( template( pageInfo ) );

... the `index` and `size` properties are set as the component's `offset` and `limit` attributes. If change the value of `index` like...

    pageInfo.attr("index", 20);

... the component's `offset` value will change and its template will update to:

    <my-paginate>Page 1</my-paginate>

### Using attribute values

You can also pass a literal string value of the attribute instead of the attribute's value looked up in the parent viewModel (similar to pass by value).

To do this in [can.stache], simply pass any value not wrapped in single brackets, and the viewModel property will be initialized to this string value:

    <my-tag title="hello"></my-tag>

The above creates a title property in the component's viewModel, whose value is a string "hello".  If instead of `title="hello"`, we use curly brackets like `title="{hello}"`, a `hello` property would be looked up in the parent viewModel.

To pass a literal string value in [can.mustache], pass a value like in `can.stache`, and set the corresponding viewModel property in the component to `@`.  For example:

    can.Component.extend({
      tag: "my-tag",
      viewModel: {
        title: "@"
      },
      template: "<h1>{{title}}</h1>"
    });
   
With source HTML like:

    <my-tag title="hello"></my-tag>
    
Results in:

    <my-tag><h1>hello</h1></my-tag>

In 3.0, `can.mustache` syntax (requiring `"@"`) will change to `can.stache`'s (not requiring `"@"`).

If the tag's `title` attribute is changed, it updates the viewModel property 
automatically.  This can be seen in the following example:

@demo can/component/examples/accordion.html

Clicking the __Change title__ button sets a `<panel>` element's `title` attribute like:

    $("#out").on("click", "button", function(){
      $("panel:first").attr("title", "Users")
      $(this).remove();
    });


## Calling methods on viewModel from events within the template

Using HTML attributes like `can-EVENT`, you can directly call a viewModel method from a template. In this example we make `<my-paginate>` elements include a next button that calls the viewModel's `next` method:

    can.Component.extend({
      tag: "my-paginate",
      viewModel: {
        offset: 0,
        limit: 20,
        next: function(context, el, ev){
          this.attr("offset", this.offset + this.limit);
        },
        page: function(){
          return Math.floor(this.attr('offset') / this.attr('limit')) + 1;
        }
      },
      template: "Page {{page}} <button can-click='next'>Next</button>"
    });

viewModel methods get called back with the current context, the element that you are listening to and the event that triggered the callback.

@demo can/component/examples/paginate_next.html

See the section on [can.view.bindings.can-EVENT] for more information.