# cobragems Proposal

Cobragems seeks to address various "hassels" of maintaining Gems in [component based rails applications](http://cbra.info). This readme is **just a proposal** for now. [Stephan Hagemann](https://github.com/shageman/) talks about some of annoyances of defining dependencies in component based rails applications in his [2015 RailsConf workshop](https://youtu.be/MsRPxS7Cu_Q?t=13m29s) and that changing Bundler to accomedate this would be a substantial chore.

Since both the Gemfile and the gemspec file are just ruby I got to thinking maybe there is some magic we can implement in those to make this a little easier.

## Maintaining paths and git locations

For any local unbuilt gem (ie any of your components) or an unpublished gem in git it is necessary to mantain those paths and git locations in *every* other component. Why not use the root application's Gemfile to look up where these resources are.

Each component Gemfile:

```
require 'cobragems/gemfile'
source 'https://rubygems.org'

gemspec

cobra_gems
```

`Cobra.gem_locations` will look up all the gems from the gemspec in the root Gemfile and if they use path, git, or svn to define the gems location then it will redefine them again.

## Maintaining exact version numbers

To ensure that each component is being tested under the same enviroment they must define the specific version of each gem they are dependant on, and each of these versions must be identical across the various components. A monkey patch of Gem::Specification and we can look up the specific version we need from the root applications Gemfile.lock.

An example component .gemspec:

```
require 'cobragems/gemspec'
$:.push File.expand_path("../lib", __FILE__)

# Maintain your gem's version:
require "user_ui/version"

# Describe your gem and declare its dependencies:
Gem::Specification.new do |s|
  s.name        = "user_ui"
  s.version     = UserUi::VERSION
  s.authors     = ["Matthew Spence"]
  s.email       = ["msaspence@gmail.com"]
  s.homepage    = "http://msaspence.com"
  s.summary     = "UI for users to manage their profiles."
  s.license     = "No license"

  s.files = Dir["{app,config,lib}/**/*", "README.md"]
  s.test_files = Dir["spec/**/*"]

  s.add_cobra_dependency "rails"

  s.add_cobra_dependency "devise"
  s.add_cobra_dependency "cancancan"
  s.add_cobra_dependency 'users'

  s.add_development_dependency 'pry'
  s.add_development_dependency 'rspec-rails'
  s.add_development_dependency 'capybara'
end

```

The gemspec still defines its dependencies but uses the root applications Gemfile and Gemfile.lock to determine the specific version it needs.

## Maintaining Development Dependencies

Development dependencies will usually be exactly the same across all compenents: I have the development tools I like and I want to use them in all of my components. Sometime components may differ on which test libraries they use but even this I feel is the exception. We can use the root applications Gemfile to not have to repeat development dependencies across components.

An example component .gemspec:

```
require 'cobragems/gemspec'
$:.push File.expand_path("../lib", __FILE__)

# Maintain your gem's version:
require "user_ui/version"

# Describe your gem and declare its dependencies:
Gem::Specification.new do |s|
  s.name        = "user_ui"
  s.version     = UserUi::VERSION
  s.authors     = ["Matthew Spence"]
  s.email       = ["msaspence@gmail.com"]
  s.homepage    = "http://msaspence.com"
  s.summary     = "UI for users to manage their profiles."
  s.license     = "No license"

  s.files = Dir["{app,config,lib}/**/*", "README.md"]
  s.test_files = Dir["spec/**/*"]

  s.add_cobra_dependency "rails"

  s.add_cobra_dependency "devise"
  s.add_cobra_dependency "cancancan"
  s.add_cobra_dependency 'users'

  s.add_cobra_development_dependancies
end

```

## Defining the root application

How do we know where the root application is? The first option is to iterate up the tree until we find a Gemfile and assume that is the root application. I feel this would work in most situations but does limit you to a two tier component structure. I'm not sure if additional tiers would every resonably required or what that would even look like. It does also mean that your components *must* be inside the root application.

An alternative would be to allow for .cobra files with various options set inside, and could define the location of the root application. Cobragems would search up the directory tree for the closest .cobra file. The simplest implementation of this would be:

```
root: .
```

or even an empty file –since `root: .` could be defined as the default, placed at the root of the application.
