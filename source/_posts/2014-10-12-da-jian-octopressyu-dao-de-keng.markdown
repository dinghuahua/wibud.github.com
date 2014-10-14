---
layout: post
title: "搭建octopress遇到的坑"
date: 2013-02-23 20:16
comments: true
categories: 鼓捣
---

如何使用octopress搭建博客这里就不赘述了，最权威的还是去看[官网教程](http://octopress.org/docs/)好了，十分的详尽。国人翻译的教程也海了去了，用某度一搜就好了。

尝鲜使用octopress搭建博客，过程一波三折，遇到了一个十分奇葩的问题，纠结了很久。

在我的ubuntu系统上将环境搭建好以后,执行`rake generate`时，出现了如下的问题。

    ## Generating Site with Jekyll
    overwrite source/stylesheets/screen.css
    Configuration from /Users/foobar/Development/project/foobar.github.com/_config.yml
    Building site: source -> public
    Liquid Exception: undefined method `gsub' for nil:NilClass in page
    /Users/foobar/Development/project/foobar.github.com/plugins/octopress_filters.rb:123:in `shorthand_url'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/context.rb:58:in `invoke'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/variable.rb:43:in `block in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/variable.rb:38:in `each'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/variable.rb:38:in `inject'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/variable.rb:38:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:94:in `block in render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `collect'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/tags/if.rb:39:in `block (2 levels) in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/tags/if.rb:37:in `each'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/tags/if.rb:37:in `block in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/context.rb:91:in `stack'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/tags/if.rb:36:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:94:in `block in render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `collect'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:82:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/template.rb:124:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/tags/include.rb:26:in `block (2 levels) in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/context.rb:91:in `stack'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/tags/include.rb:25:in `block in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/tags/include.rb:20:in `chdir'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/tags/include.rb:20:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:94:in `block in render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `collect'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:92:in `render_all'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/block.rb:82:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/template.rb:124:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/liquid-2.3.0/lib/liquid/template.rb:132:in `render!'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/convertible.rb:101:in `do_layout'
    /Users/foobar/Development/project/foobar.github.com/plugins/post_filters.rb:167:in `do_layout'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/page.rb:100:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/site.rb:204:in `block in render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/site.rb:203:in `each'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/site.rb:203:in `render'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/lib/jekyll/site.rb:41:in `process'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/gems/jekyll-0.12.0/bin/jekyll:264:in `<top (required)>'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/bin/jekyll:19:in `load'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/bin/jekyll:19:in `<main>'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/bin/ruby_noexec_wrapper:14:in `eval'
    /Users/foobar/.rvm/gems/ruby-1.9.3-p392/bin/ruby_noexec_wrapper:14:in `<main>'
    Build Failed

查了半天原因，才发现只是因为`_config.yml`文件中的`url:`没有赋值，很奇怪，在`rake setup_github_pages`时有提示你输入Repo地址，但是却没有写入这个配置文件中。之后自己写上就安了~~
