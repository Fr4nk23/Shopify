## First steps

It's very simple to get started with Liquid.  A Liquid template is rendered in
two steps: Parse and Render.  For an overview of the Liquid syntax, please read
[[Liquid for Designers]].

```ruby
@template = Liquid::Template.parse("hi {{name}}")  # Parses and compiles the template
@template.render( 'name' => 'tobi' )               # Renders the output => "hi tobi"
```

The `parse` step creates a fully compiled template which can be re-used as often
as you like.  You can store it in memory or in a cache for faster rendering
later.

All parameters you want Liquid to work with have to be passed as parameters to
the `render` method.  Liquid does not know about your Ruby local, instance, and
global variables.

## Extending Liquid

Extending Liquid is very easy.  However, keep in mind that Liquid is a young
library and requires some outside help.  If you create useful filters and tags,
please consider creating a patch and attaching it to a ticket here on this trac.

### Create your own filters

Creating filters is very easy.  Basically, they are just methods which take one
parameter and return a modified string.  You can use your own filters by passing
an array of modules to the render call like this: `@template.render(assigns,
[MyTextFilters, MyDateFilters])`.

```ruby
module TextFilter
  def textilize(input)
    RedCloth.new(input).to_html
  end
end
```

```ruby
@template = Liquid::Template.parse(" {{ '*hi*' | textilize }} ")
@template.render({}, :filters => [TextFilter])              # => "<b>hi</b>"
```

Alternatively, you can register your filters globally:

```ruby
module TextFilter
  def textilize(input)
    RedCloth.new(input).to_html
  end
end

Liquid::Template.register_filter(TextFilter)
```

Once the filter is globally registered, you can simply use it:

```ruby
@template = Liquid::Template.parse(" {{ '*hi*' | textilize }} ")
@template.render              # => "<b>hi</b>"
```

### Create your own tags

To create a new tag, simply inherit from `Liquid::Tag` and register your block
with `Liquid::Template`.

```ruby
class Random < Liquid::Tag
  def initialize(tag_name, max, tokens)
     super
     @max = max.to_i
  end

  def render(context)
    rand(@max).to_s
  end
end

Liquid::Template.register_tag('random', Random)
```

```ruby
@template = Liquid::Template.parse(" {% random 5 %}")
@template.render    # => "3"
```

### Create your own tag blocks

All tag blocks are parsed by Liquid.  To create a new block, you just have to
inherit from `Liquid::Block` and register your block with `Liquid::Template`.

```ruby
class Random < Liquid::Block
  def initialize(tag_name, markup, tokens)
     super
     @rand = markup.to_i
  end

  def render(context)
    if rand(@rand) == 0
       super
    else
       ''
    end
  end
end

Liquid::Template.register_tag('random', Random)
```

```ruby
text = " {% random 5 %} wanna hear a joke? {% endrandom %} "
@template = Liquid::Template.parse(text)
@template.render  # => In 20% of the cases, this will output "wanna hear a joke?"
```