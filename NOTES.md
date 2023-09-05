The Toopher Dev Blog is being hosted on GitHub Pages using the GitHub Jekyll integration.

Built on September 1, 2023 using Ruby 2.7.4 and Jekyll 4.3.2. See [pages build and deployment #4](https://github.com/toopher/toopher.github.io/actions/runs/6041360891). The site was using out-of-date dependencies that caused build errors, so we had to update things. The site lost some styling from before, though it is good enough.

Resources
* [GitHub Pages settings for Toopher Dev Blog](https://github.com/toopher/toopher.github.io/settings/pages)
* [GitHub Pages](https://pages.github.com/)
* [Creating a GitHub Pages site with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/creating-a-github-pages-site-with-jekyll)
* [Testing your GitHub Pages site locally with Jekyll](https://docs.github.com/en/pages/setting-up-a-github-pages-site-with-jekyll/testing-your-github-pages-site-locally-with-jekyll)
* [GitHub Pages Dependency versions](https://pages.github.com/versions/)
    * At time of publishing GitHub was using Jekyll 3.9.3 and Ruby 2.7.4

Gemfile
```
gem "jekyll", "~> 3.9.3"
```

Basic workflow
* Install Ruby 2.7.4
* Install Bundler
* Install dependencies
* Build project

```
rbenv install 2.7.4
rbenv local 2.7.4
ruby -v
gem install bundler
bundle install
bundle exec jekyll serve
open http://localhost:4000
```
