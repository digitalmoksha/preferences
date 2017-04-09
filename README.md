# Preferences 

[![Gem Version](https://badge.fury.io/rb/dm_preferences.svg)](http://badge.fury.io/rb/dm_preferences)
[![Build Status](https://travis-ci.org/digitalmoksha/preferences.svg)](https://travis-ci.org/digitalmoksha/preferences)

**Preferences** adds support for easily creating custom preferences for models.

This gem is a fork of the original [preferences gem by Aaron Pfeifer](https://github.com/pluginaweek/preferences).  It supports Rails 5 and 4.2, and all tests are working.  The gem has been renamed to [dm_preferences](https://rubygems.org/gems/dm_preferences) so as not to conflict with the original.

## Description

Preferences for models within an application, such as for users, is a pretty
common idiom.  Although the rule of thumb is to keep the number of preferences
available to a minimum, sometimes it's necessary if you want users to be able to
disable things like e-mail notifications.

Generally, basic preferences can be accomplished through simple designs, such as
additional columns or a bit vector described and implemented by preference_fu[http://agilewebdevelopment.com/plugins/preferencefu].

However, as you find the need for non-binary preferences and the number of
preferences becomes unmanageable as individual columns in the database, the next
step is often to create a separate "preferences" table.  This is where the
_preferences_ gem comes in.

_preferences_ encapsulates this design by exposing preferences using simple
attribute accessors on the model, hiding the fact that preferences are stored in
a separate table and making it dead-simple to define and manage preferences.

## Usage

### Installation

Add the following to your gem file:

    gem 'dm_preferences', '~> 1.5'

_preferences_ requires an additional database table to work.  You can generate
a migration for this table like so:

    rails generate preferences

Then simply migrate your database:

    rake db:migrate

=== Defining preferences

To define the preferences for a model, you can do so right within the model:

```ruby
class User < ActiveRecord::Base
  preference :hot_salsa
  preference :dark_chocolate, :default => true
  preference :color, :string
  preference :favorite_number
  preference :language, :string, :default => 'English', :group_defaults => {:chat => 'Spanish'}
end
```

In the above model, 5 preferences have been defined:

* hot_salsa
* dark_chocolate
* color
* favorite_number
* language

For each preference, a data type and default value can be specified.  If no
data type is given, it's assumed to be a boolean value.  If no default value is
given, the default is assumed to be nil.

### Accessing preferences

Once preferences have been defined for a model, they can be accessed either
using the accessor methods that are generated for each preference or the generic
methods that are not specific to a particular preference.

#### Accessors

There are several shortcut methods that are generated for each preference
defined on a model.  These reflect the same set of methods (attribute accessors)
that are generated for a model's columns.  Examples of these are shown below:

Query methods:

```ruby
  user.prefers_hot_salsa?         # => false
  user.preferred_language?        # => true
```

Reader methods:

```ruby
  user.prefers_hot_salsa          # => false
  user.preferred_language         # => "English"
```

Writer methods:

```ruby
user.prefers_hot_salsa = false        # => false
user.preferred_language = 'English'   # => "English"
```

#### Generic methods

Each preference accessor is essentially a wrapper for the various generic methods
shown below:

Query method:

```ruby
user.prefers?(:hot_salsa)     # => false
user.preferred?(:language)    # => true
```

Reader method:

```ruby
user.prefers(:hot_salsa)      # => false
user.preferred(:language)     # => "English"
```

Write method:

```ruby
user.write_preference(:hot_salsa, false)      # => false
user.write_preference(:language, "English")   # => "English"
```

### Accessing all preferences

To get the collection of all custom, stored preferences for a particular record,
you can access the +stored_preferences+ has_many association which is automatically
generated:

```ruby
user.stored_preferences
```

In addition to this, you can get a hash of all stored preferences *and* default
preferences, by accessing the +preferences+ helper:

```ruby
user.preferences  # => {"language"=>"English", "color"=>nil}
```

This hash will contain the value for every preference that has been defined for
the model, whether that's the default value or one that has been previously
stored.

A short-hand alternative for preferences is also available:

```ruby
user.prefs  # => {"language"=>"English", "color"=>nil}
```

### Grouping preferences

In addition to defining generic preferences for the owning record, you can also
group preferences by ActiveRecord objects or arbitrary names.  This is best shown
through an example:

```ruby
user = User.find(:first)
car = Car.find(:first)

user.preferred_color = 'red', car
# user.write_preference(:color, 'red', car) # The generic way
```

This will create a color preference of "red" for the given car.  In this way,
you can have "color" preferences for different records.

To access the preference for a particular record, you can use the same accessor
methods as before:

```ruby
user.preferred_color(car)
# user.preferred(:color, car) # The generic way
```

In addition to grouping preferences for a particular record, you can also group
preferences by name.  For example,

```ruby
user = User.find(:first)

user.preferred_color = 'red', :automobiles
user.preferred_color = 'tan', :clothing

user.preferred_color(:automobiles)  # => "red"
user.preferred_color(:clothing)     # => "tan"

user.preferences(:automobiles)      # => {"color"=>"red"}
```

### Saving preferences

Note that preferences are not saved until the owning record is saved.
Preferences are treated in a similar fashion to attributes.  For example,

```ruby
user = user.find(:first)
user.attributes = {:prefers_hot_salsa => false, :preferred_color => 'red'}
user.save!
```

Preferences are stored in a separate table called "preferences".

### Tracking changes

Similar to ActiveRecord attributes, unsaved changes to preferences can be
tracked.  For example,

```ruby
user.preferred_language               # => "English"
user.preferred_language_changed?      # => false
user.preferred_language = 'Spanish'
user.preferred_language_changed?      # => true
user.preferred_language_was           # => "English"
user.preferred_language_change        # => ["English", "Spanish"]
user.reset_preferred_language!
user.preferred_language               # => "English"
```

Assigning the same value leaves the preference unchanged:

```ruby
user.preferred_language               # => "English"
user.preferred_language = 'English'
user.preferred_language_changed?      # => false
user.preferred_language_change        # => nil
```

## Testing

You can run the tests by doing

    rake

## Original List of Resources

API

* http://rdoc.info/github/pluginaweek/preferences/master/frames

Bugs

* http://github.com/pluginaweek/preferences/issues

Development

* http://github.com/pluginaweek/preferences

Testing

* http://travis-ci.org/pluginaweek/preferences

Source

* git://github.com/pluginaweek/preferences.git

Mailing List

* http://groups.google.com/group/pluginaweek-talk


## Dependencies

* 1.5.x for Rails 5
* 1.0.x for Rails 4.2
* 0.5.6 for Rails 4.1