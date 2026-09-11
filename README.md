# Cyril Fougeray website

## Purpose

This is Cyril Fougeray's personal website, built with Jekyll and published through GitHub Pages.

## Install

```shell
# macOS: install the Ruby version used by this repository
brew install ruby@3.3
export PATH="$(brew --prefix ruby@3.3)/bin:$PATH"

# install the site dependencies locally
gem install bundler
bundle config set --local path vendor/bundle
bundle install

# preview the site
bundle exec jekyll serve
```

Then open <http://127.0.0.1:4000>. The repository targets Ruby 3.3 and tracks
`Gemfile.lock` so local builds use the same dependency set.

## Note

This website uses Jekyll theme, which is a port of [ThemeFisher's](https://themefisher.com) [Airspace template](https://themefisher.com/products/airspace-free-bootstrap-website-template/). It is released under ThemeFisher's [license](https://themefisher.com/license) , which requires attribution. Concern about the license please contact with [them](mailto:themefisher@gmail.com)
