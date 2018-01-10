---
author: jfriesen
category: practices
filename: 2017-12-18-arel.md
layout: post
tagline: Considering when to add another layer of abstraction
title: A Whirlwind through AREL
tags: 'ruby, design, arel'
---
What follows is a quick whirlwind of using AREL, as a lead up to a Ruby Code Club meetup at Notre Dame.

# What is AREL

From the [AREL Gem's README](https://github.com/rails/arel/blob/master/README.md)

> Arel Really Exasperates Logicians
>
> Arel is a SQL AST (abstract syntax tree) manager for Ruby. It
>
> * simplifies the generation of complex SQL queries, and
> * adapts to various RDBMSes.
>
> It is intended to be a framework framework; that is, you can build your own ORM with it, focusing on innovative object and collection modeling as opposed to database compatibility and query generation.

It enables:

* The 'restriction' operator (eg. `where`)
* The `SQL` `SELECT` clause is (eg. via AREL's `projection` or `#project` method)
* Comparison operators `=`, `!=`, `<`, `>`, `<=`, `>=`, `IN`
* Bitwise operators `&`, `|`, `^`, `<<`, `>>`
* Joins (both `INNER` and `OUTER`)
* Aggregate functions `AVG`, `SUM`, `COUNT`, `MIN`, `MAX`, `HAVING`
* [Common Table Expressions (CTE)](https://en.wikipedia.org/wiki/Hierarchical_and_recursive_queries_in_SQL#Common_table_expression) (e.g. temporary named result sets)
* A short circuit to write SQL literals (some things may be too complete, so you can use `Arel::SqlLiteral`)

# Advantages of AREL

* Chainable - you can reuse elements of the queries ([see example of reuse](https://github.com/ndlib/sipity/blob/47b96299eef5793556cfb45d1a9af27ea1bd992a/app/repositories/sipity/queries/processing_queries.rb#L162-L196))
  - Repeated use of a common series of JOINS
  - Encode these subqueries in methods
* ORM agnostic - you can write AREL once and rely on implementing adapters
  - Sipity used MySQL initially; I ported the code to a community project which used Postgres; The code worked right away
* Mixable - you can mix AREL with ActiveRecord's simplified query syntax, while maintaining a common ActiveRecord::Relation response object.
  - Easy to append pagination queries
  - Easy to append `#pluck` calls
  - Or other ActiveRelation methods (e.g. `#where`)

# Challenges of AREL

* Legibility can be challenging
  - AREL is an abstraction of SQL; If you aren't comfortable with SQL, AREL will exacerbate that discomfort
  - AREL builds SQL by chaining method calls (instead of a concatenating strings with SQL); Which means more parentheses
* Unions, something I've rarely used, are possible, but difficult without guidance ([some guidance with comments](https://github.com/ndlib/sipity/blob/47b96299eef5793556cfb45d1a9af27ea1bd992a/app/repositories/sipity/queries/processing_queries.rb#L64-L128))

# Working with AREL

* I spent a bit of time using Byebug and the `#to_sql` method on ActiveRecord::Relation
* Document your understanding in code comments
  - You will likely write this code once and then read it several times over; be kind and write documentation
  - I'm comfortable with SQL, perhaps fluent. AREL is like reading a newspaper in a foreign language after 2 years of high school studying that language. You can do it, but it is more challenging.
  - Because of it's "chainable" nature think about what methods are `public` and `private`; don't expose more than necessary
* Pay attention to the shape of the chained methods. Add line breaks to help the reader comprehend the AREL commands.
* For the love of all that is sacred, write tests for those queries.
  - What should be in the result set as well as what is not in the result set

# Summary

In a Rails project, when possible, keep using ActiveRecord's basic methods. They'll work across different databases. However, don't be afraid to dive into AREL to deal with a more complicated query. Take time to document your database relations (via an ERD), then document the chain of AREL methods into something meaningful.

# Examples

## A simple AREL usage from Sipity

[See source code](https://github.com/ndlib/sipity/blob/47b96299eef5793556cfb45d1a9af27ea1bd992a/app/repositories/sipity/queries/processing_queries.rb#L130-L145)
```ruby
# The following relations exist
StrategyRole.belongs_to :role
StrategyRole.belongs_to :strategy
Entity.belongs_to :strategy

# @api public
# Roles associated with the given :entity
#
# @param entity [Object] that can be converted into a Sipity::Models::Processing::Entity
# @return [ActiveRecord::Relation<Role>]
def scope_roles_associated_with_the_given_entity(entity:)
  Role.where(
    Role.arel_table[:id].in(
      StrategyRole.arel_table.project(StrategyRole.arel_table[:role_id]).where(
        StrategyRole.arel_table[:strategy_id].eq(entity.strategy_id)
      )
    )
  )
end
```

## An example of chained AREL usage from Sipity

[See source code](https://github.com/ndlib/sipity/blob/47b96299eef5793556cfb45d1a9af27ea1bd992a/app/repositories/sipity/queries/processing_queries.rb#L163-L215) *Note the reuse of `#action_registers_subquery_builder`.*

```ruby
# @api private
#
# An ActiveRecord::Relation scope that meets the following criteria:
#
# * Any user that has taken the action (or someone has taken it on their
#   behalf)
# * Any user that is part of a group that has taken the action (you know,
#   because maybe we'll allow groups to take the action)
#
# @param entity an object that can be converted into a Sipity::Models::Processing::Entity
# @param actions [Array] an array of objects that can be covnerted into a Sipity::Models::Processing::Action
#   given the entity
# @return [ActiveRecord::Relation<User>]
def users_that_have_taken_the_action_on_the_entity(entity:, actions:)
  users = User.arel_table
  group_memberships = Models::GroupMembership.arel_table

  User.where(
    users[:id].in(
      action_registers_subquery_builder(
        poly_type: User, entity: entity, actions: actions
      )
    ).or(
      users[:id].in(
        group_memberships.project(group_memberships[:user_id]).where(
          group_memberships[:group_id].in(
            action_registers_subquery_builder(
              poly_type: Sipity::Models::Group, entity: entity, actions: actions
            )
          )
        )
      )
    )
  )
end

def action_registers_subquery_builder(poly_type:, entity:, actions:)
  actors = Models::Processing::Actor.arel_table
  action_registers = Models::Processing::EntityActionRegister.arel_table
  entity = Conversions::ConvertToProcessingEntity.call(entity)
  actions = Array.wrap(actions) { |an_action| Conversions::ConvertToProcessingAction.call(an_action, scope: entity) }
  poly_type = Conversions::ConvertToPolymorphicType.call(poly_type)

  actors.project(actors[:proxy_for_id]).where(
    actors[:proxy_for_type].eq(poly_type)
  ).join(action_registers).on(
    action_registers[:on_behalf_of_actor_id].eq(actors[:id])
  ).where(
    action_registers[:strategy_action_id].in(actions.map(&:id)).
    and(action_registers[:entity_id].eq(entity.id))
  )
end
private :action_registers_subquery_builder
```
