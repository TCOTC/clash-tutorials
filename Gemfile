# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.0", ">= 7.0.1"

gem 'wdm', '>= 0.1.0' if Gem.win_platform?

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

group :test do
  gem "html-proofer", "~> 5.0"
end
