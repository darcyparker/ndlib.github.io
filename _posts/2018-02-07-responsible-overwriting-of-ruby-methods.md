---
author: jeremyf
category: practices
filename: 2018-02-07-responsible-overwriting-of-ruby-methods.md
layout: post
tagline: Preserving Extensibility
title: Responsible Overwriting of Ruby Methods
tags: 'ruby'
---

Nestled deep in a dependent gem, you have a method you need to change.
For some reason the implementation details are inadequate for your needs.

## The Setup

In the dependent gem, using pure Ruby you might see something like this:

```ruby
module Deep
  module Gem
    module Base
      def the_method_to_change
        # Business logic that isn't quite in line with your business logic.
      end
    end
  end
end

module Deep
  class Object
    # Adds the **instance methods** of the extended module as class methods to
    # this object. `Deep::Object.the_method_to_change` works but
    # `Deep::Object.new.the_method_to_change` would raise a MethodMissing Error
    extend Deep::Gem::Base
  end
end
```

In Rails world, leveraging [ActiveSupport::Concern][active_support_concern], you might see the following:

```ruby
module Deep
  module Gem
    module Base
      extend ActiveSupport::Concern
      # By convention, the methods defined in the ClassMethods submodule, will be
      # added to the class methods of the class that includes the containing module.
      module ClassMethods
        def the_method_to_change
          # Business logic that isn't quite in line with your business logic.
        end
      end
    end
  end
end

module Deep
  class Object
    # Note, we are using `include` instead of `extend`, but the effect is the
    # same; `Deep::Object.the_method_to_change` works but
    # `Deep::Object.new.the_method_to_change` would raise a MethodMissing Error
    include Deep::Gem::Base
  end
end
```

In your application you may be stuck calling `Deep::Object.the_method_to_change` yet want to update the underlying implementation. *In the real-world example, we wanted to override the behavior of [ActiveFedora's `.reindex_everything` method][active_fedora_reindex_everything].*
## Implementation

I recommend that if you want to change the method implementation, you make another module and extend that base class.
And follow the implementation pattern of the method you are overriding - use `ActiveSupport::Concern` if the upstream module uses it.

```ruby
require 'deep/object'
raise "Verify this override is still needed for non 1.0.1 versions" unless Deep::Object::VERSION == '1.0.1'
module MyNamespace
  module Overrides
    extend ActiveSupport::Concern
    module ClassMethods
      def the_method_to_change
      end
    end
  end
end

Deep::Object.include(MyNamespace::Overrides)
```

The above override has a few concepts:

1. Explicit Require: ensure the upstream class definition is loaded.
2. Provide Guidance: some guidance that the assumed override only works for version 1.0.1
3. Separate Module: to provide structure

### Explicit Require

This might be self-evident, but I want to reiterate - if you are replacing an upstream method, make sure that the method to replace is declared before you begin replacing it.

### Provide Guidance

In the days of yore, I found and patched a [Rails bug](https://github.com/rails/rails/commit/f2a0dfc2985c008a618e1616f6cf9a4c54098c33). I was working in Rails 3.0.x and wrote my module with a Rails version check. Each time I bumped the Rails version, the file would raise an exception saying "Go check if this fix has been applied." For a year or so, I walked that patch along until one day, in Rails 3.2.0, the patch was in master.

Provide context to why you are making the change; What assumptions are in play? Raise an exception if those assumptions are not valid.

* Provide guidance on how to check this assumption
* Add comments on why you are doing this
* Add information in your commit messages describing why
* **And for all that is holy and sacred, write some tests that confirm your expected behavior.**

### Separate Module

Create a separate module; This allows documentation on the nature of the module. Maybe the module contains interrelated overrides for multiple classes; Or you have a single override. Regardless, this gives a place for people to expect changes.

By mixing in another module, you preserve access to the `super` method. In the above example, I could add to the `the_method_to_change` definition a call to `super` and it call the original `Deep::Object.the_method_to_change` method.


## An Inadequate Implementation

You can see an "in the wild implementation" in CurateND. I added the [`config/initializers/active_fedora_soft_delete_monkey_patch.rb`](https://github.com/ndlib/curate_nd/blob/d93a9d91700470b10cd44539b595b1d4e3138ccf/config/initializers/active_fedora_soft_delete_monkey_patch.rb#L1) file to contain the logic for soft-deletes. The implementation details spanned two inter-related gems, but the logic in our application was inter-related. **And for those keeping score, I didn't quite follow all of my own advice.**

I'm not saying that the best place for these changes is in an initializer, but I do believe you should put them in a discoverable place (where-ever that might be in a large code-base).
## Follow-up

The collective Ruby community has spilled a lot of digital ink posting about Extend vs. Include and the nuances. Some posts to consider.

* [A 2009 RailsTips](http://www.railstips.org/blog/archives/2009/05/15/include-vs-extend-in-ruby/)
* [A 2013 AppFolio Engineering](http://engineering.appfolio.com/appfolio-engineering/2013/06/17/ruby-mixins-activesupportconcern)

And a personal favorite by [Jay Fields for alternatives ways to redefine Ruby methods](http://blog.jayfields.com/2008/04/alternatives-for-redefining-methods.html). Seriously this blog post was what hooked me on Ruby; Methods are detachable and re-attachable lambdas.


[active_support_concern]:http://api.rubyonrails.org/v5.1/classes/ActiveSupport/Concern.html
[active_fedora_reindex_everything]:https://github.com/samvera/active_fedora/blob/be1ec093ecf0c6ec4ce4e60b7f367bd71a06029b/lib/active_fedora/indexing.rb#L95
