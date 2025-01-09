# Handcuffs

[![Test](https://github.com/procore-oss/handcuffs/actions/workflows/test.yaml/badge.svg?branch=main)](https://github.com/procore-oss/handcuffs/actions/workflows/test.yaml)
[![Gem Version](https://badge.fury.io/rb/handcuffs.svg)](https://badge.fury.io/rb/handcuffs)
[![Discord](https://img.shields.io/badge/Chat-EDEDED?logo=discord)](https://discord.gg/PbntEMmWws)

Handcuffs provides an easy way to run migrations in phases in your [Ruby on Rails](https://rubyonrails.org/) application.

<<<<<<< Updated upstream
To configure, first create a handcuff initializer and define a configuration

```ruby
# config/initializers/handcuffs.rb

Handcuffs.configure do |config|
  config.phases = [:pre_restart, :post_restart]
end
```

Then call `phase` from inside your migrations

```ruby
# db/migrate/20160318230933_add_on_sale_column.rb

class AddOnSaleColumn < ActiveRecord::Migration

  phase :pre_restart

  def up
    add_column :products, :on_sale, :boolean
  end

  def down
    remove_column :products, :on_sale
  end

end
```

```ruby
# db/migrate/20160318230988_add_on_sale_index

class AddOnSaleIndex < ActiveRecord::Migration

  phase :post_restart

  def up
    add_index :products, :on_sale, algorithm: :concurrently
  end

  def down
    remove_index :products, :on_sale
  end

end
```

You can then run your migrations in phases using

```bash
rake 'handcuffs:migrate[pre_restart]'
```

or

```bash
rake 'handcuffs:migrate[post_restart]'
```

You can run all migrations using

```bash
rake 'handcuffs:migrate[all]'
```

This differs from running `rake db:migrate` in that migrations will be run in the _order that the phases are defined in the handcuffs config_. By default, a phase is assumed to depend on all phases named before it. Phase dependencies can be customized by declaring a dependency graph explicitly:

```ruby
# config/initializers/handcuffs.rb

Handcuffs.configure do |config|
  config.phases = {
    # Prevent running post_restart migrations if there are outstanding
    # pre_restart migrations
    post_restart: [:pre_restart],
    # Require pre_restarts before data_migrations, but do not enforce ordering
    # between data_migrations and post_restarts
    data_migrations: [:pre_restart],
    # pre_restarts have no prerequisite phases
    pre_restart: []
  }
end
```

The phase order of `handcuffs:migrate[all]` will satisfy these dependencies, but is otherwise unspecified.

If you run a handcuffs rake task and any migration does not have a phase defined, an error will be raised before any migrations are run. To prevent this error, you can define a default phase for migrations that don't define one.

```ruby
# config/initializers/handcuffs.rb

Handcuffs.configure do |config|
  config.phases = [:pre_restart, :post_restart]
  config.default_phase = :pre_restart
end
```
=======
1. Defined a set of named phases and the order in which they should be run
2. Tag migrations with phase names
3. Execute migrations in batches or in series by phase name
>>>>>>> Stashed changes

## Installation

Add this line to your application's Gemfile:

```ruby
gem 'handcuffs'
```

And then execute:

```bash
bundle
```

Or install it globally on the current system using:

```bash
gem install handcuffs
```

## Usage

### Configuration

Create a handcuffs initializer and define the migration phases in the order in which they should be run. You should also define a default phase for pre-existing migrations or if you want the option not to tag every migration.


The most basic configutation is an array of phase names:

```ruby
# config/initializers/handcuffs.rb

Handcuffs.configure do |config|
  # pre_restart migrations will/must run before post_restart migrations
  config.phases = [:pre_restart, :post_restart]
  config.default_phase = :pre_restart
end
```

If you have more complex or asynchrous workflows, you can use a explicit hash notation that allows pre-requisite stages to be specified explicitly in order to define dependencies:

```ruby
# config/initializers/handcuffs.rb

Handcuffs.configure do |config|
  config.phases = {
    # Prevent running post_restart migrations if there are outstanding
    # pre_restart migrations
    post_restart: [:pre_restart],
    # Require pre_restarts before data_migrations, but do not enforce ordering
    # between data_migrations and post_restarts
    data_migrations: [:pre_restart],
    # pre_restarts have no prerequisite phases
    pre_restart: []
  }
end
```

In order to validate the configuration (especially to prevent circular dependencies) it is reccomended that you check the phase configuration after any changes using the rake task:

```ruby
rake handcuffs:phase_order
```


### Tagging Migrations

Once you have configured the order of each phase and their dependencies, you should then define a `phase` inside each of your migrations, e.g.

```ruby
# db/migrate/20240318230933_add_on_sale_column.rb

class AddOnSaleColumn < ActiveRecord::Migration[7.0]

  phase :pre_restart

  def change
    add_column :products, :on_sale, :boolean
  end
end
```

```ruby
# db/migrate/20160318230988_add_on_sale_index

class AddOnSaleIndex < ActiveRecord::Migration[7.0]

  phase :post_restart

  def change
    add_index :products, :on_sale, algorithm: :concurrently
  end
end
```

### Running Migrations

Once migrations are properly tagged, you can then run your migrations in phases using the handcuffs rake tasks and passing in the phase you want to run:

```bash
rake 'handcuffs:migrate[pre_restart]'
```

or

```bash
rake 'handcuffs:migrate[post_restart]'
```

Note, though, that if there are any pre-requisite phases that have migrations (e.g. `phase :pre_restart`) that have not yet been run, trying to run a dependent phase (e.g. `phase: post_restart`) will raise a `HandcuffsPhaseOutOfOrderError`.

You can run all migrations in all phases order using

```bash
rake 'handcuffs:migrate[all]'
```

This differs from running `rake db:migrate` in that migrations will be run in batches corresponding to the _order that the phases are defined in the handcuffs config_. Again, you can use `rake handcuffs:phase_order` to preview the order ahead of time.


Finally, you can still run `rake db:migrate` at any time to run migrations in the standard order based on their timestamped file name. Handcuffs phases will be ignored.


## Contributing

Bug reports and pull requests are welcome on GitHub at <https://github.com/procore-oss/handcuffs>. This project is intended to be a safe, welcoming space for collaboration, and contributors are expected to adhere to the [Contributor Covenant](http://contributor-covenant.org) code of conduct.


## Running Tests Locally

The specs for handcuffs are in the dummy application at `/spec/dummy/spec`. The spec suite requires PostgreSQL. To run it you will have to set the environment variables `POSTGRES_DB_USERNAME` and `POSTGRES_DB_PASSWORD`. You can then run the suite using `rake spec`

## License

The gem is available as open source under the terms of the [MIT License](http://opensource.org/licenses/MIT).

## About Procore

<img
  src="https://www.procore.com/images/procore_logo.png"
  alt="Procore Logo"
  width="250px"
/>

Handcuffs is maintained by Procore Technologies.

Procore - building the software that builds the world.

Learn more about the #1 most widely used construction management software at [procore.com](https://www.procore.com/)
