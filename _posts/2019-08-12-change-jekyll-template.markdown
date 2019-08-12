---
layout: post
comments: true
title:  "更换jekyll模板 解决找不到Gemfile"
date:   2019-08-12 14:22:24 +0800
tags: jekyll Gemfile 原创
lang: zh
---

模板：
- 博客：[https://huxpro.github.io](https://huxpro.github.io/)
- github：[https://github.com/Huxpro/huxpro.github.io/](https://github.com/Huxpro/huxpro.github.io/)

git clone 他的仓库，然后在该目录执行`bundle install`或者`bundle exec jekyll serve`发现会报错，~~执行`jekyll serve`会报部分错，但是跑得起来，然而跑起来之后又会报各种404错误，~~我写这个的时候打算复现那个404错误发现又不报错了

```
d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ bundle install
Could not locate Gemfile

d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ bundle exec jekyll serve
Could not locate Gemfile or .bundle/ directory

d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ
```

报错提示找不到Gemfile，所以要想办法整一个，在该项目目录下执行`jekyll new . --force`

```
d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ jekyll new . --force
Running bundle install in d:/personal_blog/Huxpro...
  Bundler: Fetching gem metadata from https://rubygems.org/...........
  Bundler: Fetching gem metadata from https://rubygems.org/.
  Bundler: Resolving dependencies...
  Bundler: Using public_suffix 3.1.1
  Bundler: Using addressable 2.6.0
  Bundler: Using bundler 2.0.2
  Bundler: Using colorator 1.1.0
  Bundler: Using concurrent-ruby 1.1.5
  Bundler: Using eventmachine 1.2.7 (x86-mingw32)
  Bundler: Using http_parser.rb 0.6.0
  Bundler: Using em-websocket 0.5.1
  Bundler: Using ffi 1.11.1 (x86-mingw32)
  Bundler: Using forwardable-extended 2.6.0
  Bundler: Using i18n 0.9.5
  Bundler: Using rb-fsevent 0.10.3
  Bundler: Using rb-inotify 0.10.0
  Bundler: Using sass-listen 4.0.0
  Bundler: Using sass 3.7.4
  Bundler: Using jekyll-sass-converter 1.5.2
  Bundler: Using ruby_dep 1.5.0
  Bundler: Using listen 3.1.5
  Bundler: Using jekyll-watch 2.2.1
  Bundler: Using kramdown 1.17.0
  Bundler: Using liquid 4.0.3
  Bundler: Using mercenary 0.3.6
  Bundler: Using pathutil 0.16.2
  Bundler: Using rouge 3.8.0
  Bundler: Using safe_yaml 1.0.5
  Bundler: Using jekyll 3.8.6
  Bundler: Using jekyll-feed 0.12.1
  Bundler: Using jekyll-seo-tag 2.6.1
  Bundler: Using minima 2.5.0
  Bundler: Using thread_safe 0.3.6
  Bundler: Using tzinfo 1.2.5
  Bundler: Using tzinfo-data 1.2019.2
  Bundler: Using wdm 0.1.1
  Bundler: Bundle complete! 6 Gemfile dependencies, 33 gems now installed.
  Bundler: Use `bundle info [gemname]` to see where a bundled gem is installed.
New jekyll site installed in d:/personal_blog/Huxpro.

d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ
```

这时候项目目录会变成这样：

![项目目录图片](https://s2.ax1x.com/2019/08/12/ezrrOU.png)

将`.gitignore`、`_config.yml`、`404.html`替换为原版，执行`git log -1`查看最新的commit号是多少，然后执行`git checkout <commit号的前几位> <文件名>`

```
d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ git log -1
commit 3a4bf1c7ceb2c98971b7091b1dc485175dc897f6 (HEAD -> master, origin/master)
Author: Xuan Huang <huxpro@gmail.com>
Date:   Tue Jul 23 01:40:03 2019 -0700

    [self] update

d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ git checkout 3a4bf1c7ce .gitignore _config.yml 404.html

d:\personal_blog\Huxpro (master -> origin) (hux-blog@1.7.0)
λ
```

找到原版`_config.yml`中`plugins: [jekyll-paginate]`这行（我复制这个模板的时候这里只写了这一个plugin，如果后边有新增的就按照同样的方法更改Gemfile就可以了），接着打开生成的`Gemfile`，然后增加一行`gem "jekyll-paginate"`

![更改后的Gemfile](https://s2.ax1x.com/2019/08/12/ezh6vd.png)

删除多余的markdown文件，这里是`about.md`、`index.md`

现在目录中只有Gemfile和Gemfile.lock是新增的了

完成了，现在可以执行`bundle install`和`bundle exec jekyll serve`，后边就是将原作者的内容更换为自己的内容了。

