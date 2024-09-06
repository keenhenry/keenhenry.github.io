# frozen_string_literal: true

source "https://rubygems.org"

gem "jekyll-theme-chirpy", "~> 7.1"

gem "html-proofer", "~> 5.0", group: :test

# If you want to use GitHub Pages, remove the "gem "jekyll"" above and
# uncomment the line below. To upgrade, run `bundle update github-pages`.
#gem "github-pages", group: :jekyll_plugins

platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.1.1", :platforms => [:mingw, :x64_mingw, :mswin]

# kramdown v2 ships without the gfm parser by default. If you're using
# kramdown v1, comment out this line.
gem "kramdown-parser-gfm"

# The following gems were added to silence Jekyll build/runtime warnings with a newer version of Ruby (>3.3)
gem "logger", "~> 1.6.1"
gem "csv", "~> 3.3"
gem "base64", "~> 0.2"
