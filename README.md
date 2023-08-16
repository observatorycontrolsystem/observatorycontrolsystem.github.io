# observatorycontrolsystem.github.io
Open source software for an API-driven observatory

This is a Jekyll project hosted on GitHub pages. Follow the instructions [here](https://jekyllrb.com/docs/) for local
development. The web page is deployed on any push to master.

## Local development instructions:
1. Make sure ruby is installed on your system (may need ruby-devel as well)
2. Install gems: `gem install jekyll bundler bulma-clean-theme`
3. Add the gems to bundler: `bundle init; bundle add jekyll bulma-clean-theme` (may need to add as root, and add `jekyll-feed` as well)
4. Run `bundle exec jekyll serve --livereload` to start up a local version of the site.
