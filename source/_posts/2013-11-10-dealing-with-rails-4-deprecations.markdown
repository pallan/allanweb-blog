---
layout: post
title: "Dealing with Rails 4 Deprecations"
date: 2013-11-10 11:21
comments: true
categories: 
categories: [ruby, rails]
published: true
---

At my place of employment we have a very large Rails application that
runs many parts of the business. The app was started on Rails 2.0 and went
into production under Rails 2.3. It was only recently that we were able
to get it updated to Rails 3.2 and Ruby 2.0. We the took a shot at getting 
it running under Rails 4. It took a few hours to get the site to load and
the tests to run. We triggered a build in Jenkins proceeded to watch the 
console drown in DEPRECATION warnings.

Most of these deprecations were directly related to the eventual
removal of the old ActiveRecord finder syntax to the newer Arel syntax
initially adopted in Rails 3. Here are some examples,

```
DEPRECATION WARNING: Calling #find(:all) is deprecated. Please call #all directly instead. ...
DEPRECATION WARNING: Calling #find(:first) is deprecated. Please call #first directly instead.
DEPRECATION WARNING: Calling #scope or #default_scope with a hash is deprecated. Please use a lambda containing a scope.
DEPRECATION WARNING: Passing options to #find is deprecated. Please build a scope and then call #find on it
DEPRECATION WARNING: Relation#all is deprecated. If you want to eager-load a relation, you can call #load
DEPRECATION WARNING: Relation#first with finder options is deprecated. Please build a scope and then call #first on it instead.
DEPRECATION WARNING: The following options in your Model.has_many :notes declaration are deprecated: ...
```

## Enter Arel Converter

The [arel_converter](https://github.com/pallan/arel_converter) gem is a simple command line utility that uses parses
Ruby files to find code whose syntax will be deprecated under Rails 4
and converts it to be Rails 4 compatible. The converters use Ruby2Ruby 
and RubyParser to convert to and translate s-expressions. This ensures 
the code is converter back into valid Ruby code. Fancy regex and string
parsing just doesn't work as well.

Presently, there are 3 converters,

### Scope

Converts 'scope' statements into the updated Rails 4 syntax, i.e.
wrapped in a lambda, with pre-Arel options converted to Arel

```ruby
scope :active, :conditions => {:active => true}, :order => 'created_at'
# becomes
scope :active, -> { where(active: true).order('created_at') }
```

### Associations

Updates associations to use the updated Rails 4 syntax, i.e. wrapped in
lambdas where necessary. Association types handled are,

* belongs_to
* has_one
* has_many
* has_and_belongs_to_many

```ruby
has_many :articles, :class_name => "Post", :order => 'updated_at DESC'
# becomes
has_many :articles, -> { order('updated_at DESC') }, class_name: "Post"
```

### Finders

Updates old ActiveRecord (pre Rails 3) syntax into Arel syntax of
chained Relations. This handles the following, including chained calls,

* Object.find(:all
* Object.find(:first
* Object.find.*:conditions
* Object.all(
* Object.first(

```ruby
Model.find(:all, :conditions => ['name = ?', params[:term]], :limit => 5)
# becomes
Model.where('name = ?', params[:term]).limit(5)
```

Fortunately our application has a wide variety of calls to each of the
above possibilities. Many of these variations ended up in the translator
tests. 

## Notes/Warnings
As with any software that could potentially update a wide swath of your
codebase in one (terrible) moment, it is recommended that you are
operating under some form of source control or have a reliable backup
*BEFORE* running the script.

The converters don't play well with multiline code blocks. This means
that the parser will encounter a Exception (likely SyntaxError) when
processing the first line. This is because the matching code is a simple
grep of the file. Therefore, it would only grab the first line of a
multiline statement. The good news is that these exceptions will show up
in the results.

## Check it out!

The gem is available at [rubygems.org](https://rubygems.org/gems/arel_converter) and the source is available on
[Github](https://github.com/pallan/arel_converter). Hopefully, it can
save you or your team some time and pain of trolling through thousands
of lines of code to get ready for Rails 4.1.a

