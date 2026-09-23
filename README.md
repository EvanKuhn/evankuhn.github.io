# My Blog
About tech. Mostly to record things I learn, share a few things I know, and hopefully help people solve problems.

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
