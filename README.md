# ReBolt - Install Puppet Bolt from RubyGems

## What

Installs [Puppet Bolt](https://github.com/puppetlabs/bolt) [gem](https://rubygems.org/gems/bolt)
and the [Puppet Forge](https://forge.puppet.com) modules it requires.

## Why
Puppet makes it hard to run Bolt without using their binary packages. They provide zero
documentation on how to do so, but have plenty of warnings on their website not to try it. I wanted
to be able to run Bolt on my Mac Studio without using Rosetta, both in macOS and in a Linux VM.
With the official binary packages none of this is possible. But come on, it's open source and as I
noted above [Bolt is on RubyGems](https://rubygems.org/gems/bolt)!

Puppet's official binary packages consistantly lag in platform support. They have no support for
arm64, except under Rosetta on macOS, and no arm64 at all on Linux. You may also want to run Bolt
on a more recent Ruby version than the vendored version that Puppet ships, or just want to manage
your own copy rather than depending on their packages. Or maybe you want to run Bolt on a
Raspberry Pi!

Finally, the binary package makes it hard to find where relevant files (like the built in modules)
are and understand how Bolt actually works. Installing Bolt from RubyGems and Puppet Forge, should
help demystify things, even if you still stick to the official binaries for production use.

## Installation
* Install ruby:
  * `brew install ruby` / `apt-get install ruby` / etc.
    * __Do not use the ruby shipped by Apple with macOS, it's far too old to work.__
  * Confirm that bundler is installed (it's included with rubies from Homebrew)
    * `bundle --version`
    * If not, install it from your OS package manager (`apt-get install ruby-bundler`, etc.)
* Clone this repo, cd into it.
* `bundle install`
* `rake`

## Updating
* This repo: `git pull` (I'll consider adding tagged releases and packages if this gets popular,
  but for now it's just getting started)
* Puppet Bolt gem and its gem dependencies: `bundle update --all`
* Puppet Forge modules: `rake`

Important: after updating the Bolt gem with `bundle`, you must run `rake` again or you may be left
with out of date Forge modules.

## Troubleshooting:

If you see this warning, it means that Bolt's module dependencies are missing.
```
Bolt might be installed as a gem. To use Bolt reliably and with all of its
dependencies, uninstall the 'bolt' gem and install Bolt as a package:
https://puppet.com/docs/bolt/latest/bolt_installing.html
```

Try running `rake setup:modules` or, maximally, `rake clean; rake setup`.

You don't have to take my word that running Bolt this way is safe. Take a look at the [code that
prints this warning](https://github.com/puppetlabs/bolt/blob/42ec31cc298b981f1e853820caf701e8a5573a8e/lib/bolt/cli.rb#L774-L810).
It's just checking for missing modules.
