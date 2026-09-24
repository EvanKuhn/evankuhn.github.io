# My Tech Blog
My blog about tech and software engineering. 

Setup:

```bash
# Install rbenv and ruby
brew install rbenv
echo 'eval "$(rbenv init - bash)"' >> ~/.profile
rbenv install

# Test ruby version
cat .ruby-version
ruby --version
```

To run locally:

```bash
bundle install
bundle exec jekyll build
bundle exec jekyll serve
```

Jekyll will dynamically rebuild the site as you work, which is very nice for local development.
