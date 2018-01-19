---
author: jfriesen
category: practices
filename: 2018-01-19-redefining-instance-methods-at-runtime.md
layout: post
tagline: Or With Great Power Comes Great Responsibility
title: Redefining Instance Methods at Runtime
tags: 'ruby'
---

## TL;DR

In Ruby, each object is a class, and a class is an object. You can redefine one object's method and not affect other objects of that class.

## The Task

At one point, we loaded into our system a test object that violated our data model. We performed the load via an alternate method as a bit of an experiment.

This object was stubborn, refusing any basic attempts at destruction.

I ssh-ed to our machine, loaded up the ruby console, and ran the following:

```ruby
rails $ gf = GenericFile.find('my-id')
rails $ gf.destroy
=> NoMethodError: undefined method 'representative' for nil:NilClass
```

I tried the usual, without knowing the code: `gf.representative = nil`. That didn't work. It turns out the representative was an alias for the identifier. So I looked at the validation logic:

```ruby
def check_and_clear_parent_representative
  if batch.representative == self.pid
    batch.representative = batch.generic_file_ids.select{|i| i if i != self.pid}.first
    batch.save!
  end
end
```

Not wanting to spend too much effort on this, I brought out one of Ruby's developer weapons.

```ruby
def gf.check_and_clear_parent_representative; true; end
```

Then I ran `gf.destroy` and "Poof!" the object passed validation and was destroyed.

## Explanation

For any object in Ruby you can redefine, in isolation, its instance methods. (eg. `def my_instance.the_method`) This is because each Ruby object has its own singleton class; in essence of copy of the object's class.

See [Peter J Jones's "Understanding Ruby Singleton Class" blog post](https://www.devalot.com/articles/2008/09/ruby-singleton) for further details.

I don't use this tool much, in fact I recommend that you use it only in the context of a one-off solution. This can confound other developers and [invalidates the global method cache](https://mensfeld.pl/2015/04/ruby-global-method-cache-invalidation-impact-on-a-single-and-multithreaded-applications/). In the case of the former, you are increasing the cognitive density of the application. In the case of the latter, you are decreasing performance.

But sometimes, you need to bring a dangerous tool to move past an arbitrary barrier. But know the consequence.
