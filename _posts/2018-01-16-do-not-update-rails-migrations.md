---
author: jeremyf
category: practices
filename: 2018-01-16-do-not-update-rails-migrations.md
layout: post
tagline: Unless You Like Dropping Databases
title: Do Not Update Rails Migrations
tags: 'ruby, migrations, database'
---

## TL;DR

Once you have run a Rails migration via `rake db:migrate` or `rails db:migrate`, do not modify that migration.

## Discussion

In a [pull request review for Curax](https://github.com/ndlib/curax/pull/10), my colleague asked:

> This is a very basic question, but I'm kind of learning here..
> Why would you prefer adding a new db/migrate file to remove index instead of updating the older files..
> Like in this case updating https://github.com/ndlib/curax/blob/master/db/migrate/20180110162655_update_user_for_cas_authentication.rb#L4 would take the index off :users field

My response:

> Regarding creating a new migration vs. re-using an existing one. Once a migration has been applied (via rake db:migrate) it modifies the schema and never runs again (except if you drop the database and run rake db:migrate again). If I were to modify the migration, which had already run, that modification would not be applied.
>
> If you look at
>
> curax/db/schema.rb
>
> [Line 13 in ndlib/curax/0b10717](https://github.com/ndlib/curax/blob/0b107178cd2502ae7744d619e7f8ba812bec1213/db/schema.rb#L13)
> ```ruby
> ActiveRecord::Schema.define(version: 20180110162655) do
> ```
>The version is "20180110162655", which is the prefix to the migration file you linked to. This indicates that, in all likelihood, all migrations that were written prior to 20180110162655 have been applied (the migrations that have been run are stored in a schema table inside the application's database).

## Further Discussion

Confession, I have done exactly what I say not to do. I have modified an existing migration after I have run `db:migrate`.

In those cases, I will:

1. Run a `db:rollback` command
1. Revert changes to `db/schema.rb`
1. Modify the migration
1. Run `db:migrate`

But I will only do this if I am working in a branch that has not been pushed and merged upstream. Once a migration hits origin/master, consider it immutable.
