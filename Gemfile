# Local preview only. GitHub Pages builds this theme's consumers with its own
# pinned toolchain, which the github-pages gem reproduces: Jekyll 3.9 and
# jekyll-sass-converter 1.5 (Ruby Sass 3). That is why the SCSS below _sass/
# stays on `@import` — `@use` would build here on a modern Sass and then fail
# on GitHub Pages.
source "https://rubygems.org"

gem "github-pages", group: :jekyll_plugins

# Ruby no longer ships these as default gems, and Jekyll 3.9 predates that.
# webrick left in Ruby 3.0, the rest in 3.4; without it `jekyll serve` cannot
# start at all, which is what script/server runs.
gem "base64"
gem "bigdecimal"
gem "csv"
gem "logger"
gem "ostruct"
gem "webrick"
