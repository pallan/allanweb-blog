---
layout: post
title: "Custom Git Commands as a Rubygem"
date: 2013-09-15 19:42
comments: true
categories: [ruby, git, rubygems]
published: false
---

[Git](http://git-scm.org) is great. Back in the day, I was dedicated to Subversion. When the development team I was on wanted to switch over to git, I really wasn't very thrilled. They dragged me kicking and screaming into the future and I'm not to big to admit that I was wrong. I haven't once been tempted to look back.

One of greatest benefits of git is its flexibility. When learning git for the first time most users learn about the ability to alias commands.

```
$ git config alias.br branch
```

This is great for those who cut their source control teeth on another system as it allows you to alias all the commands you were familiar with to work the their git counterparts.

Unless you are a shell script programming guru the limitations of aliasing can become apparent fairly quickly. And that's okay, because git makes it very easy to add your own custom commands for those more complicated tasks.  The simple example I am going to use is a git command to bump the version of the current repository. Unsurprisingly, the name of this command will be:

```
$ git bump
```

All this command will do it look at the last annotated version tag applied to the repository and increment it. We will assume the tag will be in the format "v#.#.#" (e.g. v2.3.4). Got it? Okay, let's get started.

### Step 1: Create a rubygem

There are countless articles/tutorials out there explaining how to create a new rubygem. There are also a plethora of tools to help you do it (along with the arguments about why you should use one and not the other). Personally, I like the simplicity of using bundler to create a new gem. I am not going to get into all the details about how to use bundler to do this because a [guide](https://github.com/radar/guides/blob/master/gem-development.md) already exists.

### Step 2: "Bump" Command

Next, lets build our Ruby class that will do the work of parsing the existing version tag and creating the new one.

```ruby
class BumpVersion
  
  attr_accessor :major, :minor, :patch
  
  def call
    parse_version
    increment_patch_level
  end
  
  def parse_version
    v = /v(\d+)\.(\d+)\.(\d+)/.match(current_version)
    self.major, self.minor, self.patch = v[1], v[2], v[3]
  end
  
  def current_version
    `git describe --abbrev=0`
  end
  
  def increment_patch_level
    "v#{major}.#{minor}.#{patch.to_i + 1}"
  end
  
end
```


### Step 3: Ruby Command

Ruby at its heart is a scripting language. Creating shell commands using Ruby is very easy and soooo much nicer than diving into bash, IMO. Let's create a quick shell command in Ruby we can use to call our BumpVersion class.

```ruby
#!/usr/bin/env ruby

$LOAD_PATH.unshift File.join(File.dirname(__FILE__), '..', 'lib')

require 'bump_version'

exit BumpVersion.new(STDIN, STDOUT, *ARGV).call
```


### Step 4: Tests

### Step 5: Install & Go

### Next Steps

As I am sure you can see there is plenty of room for improvement over this initial implementation. What happens if the last annotated tag is not a version tag? What if there is no annotated tags? How do I check to make sure the correct number is being set? There are a variety of command options you could add to make this far more useful. For example, `-p` or `--push` to push the tag to a remote, `--minor` or `--major` to force incrementing something other than the patch level or setting the version number explicitly:

```
$ git bump v8.3.234
```

I will leave those options as a exercise to the reader. If you take up the challenge make sure you look into the core Ruby class [OptParse](http://ruby-doc.org/stdlib-2.0.0/libdoc/optparse/rdoc/OptionParser.html) to ease dealing with all those flags and switches.

### Acknowledgements

The example gem is heavily inspired by [trydionel's git-pivotal gem](https://github.com/trydionel/git-pivotal). My first git commands were created for integration with [Redmine](http://www.redmine.org/) and the aforementioned gem helped steer me in the right direction to get that built.


### TODO

* create the binary command
* create the tests
* install the gem
* demo it working

