# Cascalog Documentation

This is a documentation site for [Cascalog](http://www.cascalog.org) hosted and auto-generated directly from this repository. It uses Jekyll site generator with Markdown as the article markup language. Any changes made to the `gh-pages` branch here will be live immediately.

## Contributing

Guides and articles are in the [/articles](/articles) directory. You can clone this repository and then add or edit articles there, then send a pull request.

### Quick Editing

For quick editing of the content, you can edit files in the [/articles](/articles) directory using Github web interface. Github would then take care of forking and sending pull request automatically for you.

## Development

For major changes, it is recommended to setup a local environment with Jekyll if you want to view your changes as a web site locally.

### Install Dependencies

With Ruby Bundler:

    bundle install --binstubs

#### How to run a development server

    ./bin/jekyll serve -w --trace

then navigate to [localhost:4000](http://localhost:4000)

#### How to regenerate the site

    ./bin/jekyll build