# frozen_string_literal: true

source "https://rubygems.org"

# gem "rails"

# gem "jekyll", "~> 4.3"

gem "github-pages", "~> 228", group: :jekyll_plugins

gem "jekyll-octicons", "~> 14.2"

install_if -> { ENV["GITHUB_ACTIONS"] != "true" } do
	gem "tzinfo-data", "~> 1.2023"
	gem "webrick", "~> 1.8"
end
