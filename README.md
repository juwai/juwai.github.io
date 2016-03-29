# Juwai DEV Blog

Our awesome blog for Juwai IT team, based on [Jekyll](https://jekyllrb.com/), hosted on Github pages.

### Install

1. Make sure [ruby](https://www.ruby-lang.org/) and [gem](https://rubygems.org/) has installed in your host.
1. `$ [sudo] gem install jekyll github-pages`.
1. Clone repo to you local.
1. Goto the folder and `$ jekyll serve`.
1. :point_right: `http://localhost:4000`.

### Troubleshoot

If you run into the following error:
➜  juwai.github.io git:(master) sudo gem install jekyll github-pages
Password:
ERROR:  Could not find a valid gem 'jekyll' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)
ERROR:  Could not find a valid gem 'github-pages' (>= 0), here is why:
          Unable to download data from https://rubygems.org/ - Errno::ECONNRESET: Connection reset by peer - SSL_connect (https://rubygems.org/latest_specs.4.8.gz)

please do the following to fix:
➜  juwai.github.io git:(master) gem sources --remove https://rubygems.org/
➜  juwai.github.io git:(master) gem sources -a https://ruby.taobao.org/
➜  juwai.github.io git:(master) gem sources -l
*** CURRENT SOURCES ***
https://ruby.taobao.org
➜  juwai.github.io git:(master) sudo gem install rack
➜  juwai.github.io git:(master) sudo gem install jekyll github-pages
