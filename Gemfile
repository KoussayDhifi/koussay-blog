# frozen_string_literal: true

source "https://rubygems.org"

# The main Jekyll gem required to build the site
gem "jekyll", "~> 4.2"

# Optional: Jekyll theme if you're using one like Chirpy
gem "jekyll-theme-chirpy"

# Optional: For testing your HTML (like checking broken links)
gem "html-proofer", "~> 5.0", group: :test

# Platform-specific gems for Windows
platforms :mingw, :x64_mingw, :mswin, :jruby do
  gem "tzinfo", ">= 1", "< 3"
  gem "tzinfo-data"
end

gem "wdm", "~> 0.2.0", :platforms => [:mingw, :x64_mingw, :mswin]
