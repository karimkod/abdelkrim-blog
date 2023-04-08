source "https://rubygems.org"

#gem "jekyll", "~> 3.5"

gem "minima", "~> 2.5"
gem "github-pages", group: :jekyll_plugins

group :jekyll_plugins do
  # gem "jekyll-feed", "~> 0.12"
  gem "jekyll-seo-tag"
end

install_if -> { RUBY_PLATFORM =~ %r!mingw|mswin|java! } do
  gem "tzinfo", "~> 1.2"
 gem "tzinfo-data"
end

# Performance-booster for watching directories on Windows
gem "wdm", "~> 0.1.1", :install_if => Gem.win_platform?


gem "webrick", "~> 1.8"
