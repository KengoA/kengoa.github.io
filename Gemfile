source "https://rubygems.org"

# GitHub Pages pins Jekyll + Rouge + supported plugins
gem "github-pages", "~> 231", group: :jekyll_plugins

group :jekyll_plugins do
  gem "jekyll-feed"
end

# Required for Ruby 3+ local builds
gem "webrick", "~> 1.8"

# Windows / JRuby support
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

# Windows filesystem watcher
gem "wdm", "~> 0.1.1", platforms: [:mingw, :x64_mingw, :mswin]

# JRuby HTTP parser compatibility
gem "http_parser.rb", "~> 0.6.0", platforms: [:jruby]
