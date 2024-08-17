---
title: "Jekyll之Rake脚本一键生成文章模版"
layout: post
category: post
tags: 
---

最近在搭建博客的时候才接触到Jekyll，当然之前也搭建过Hexo的博客。但是由于Jekyll是Github Page默认搭配，所以自己又折腾着搭建了一个Jekyll的博客。如果是使用Jekyll来搭建博客，那就不得不说说Rake。
##### Rake的意思是Ruby Make,一个用ruby开发的代码构建工具.
Rake其实就是Ruby的一个工程构建工具，诸如Makefile、CMake, Qt的qmake, Java的ant，以及Gradle一样，Rakefile的功能也非常强大。简单说，Rakefile其实就是Ruby语法的makefile.
### 官方说明有如下优点：
* Ruby语法
* 可以设定task的依赖
* 支持patterns的规则
* 灵活的FileList类, 行为像array, 但是可以方便的操作文件名和路径
* 有一个预先包装好的库, 可以方便的实现类似build tarball和发布到ssh网站等功能
* 支持并行task

在Jekyll博客搭建完成后，在写博客的过程中发现每次创建博文的时候很不方便，在网上查找下相关方面的资料，发现其实是可以使用Rake脚本来实现的。

#### [关于Rake](https://github.com/ruby/rake)

#### 使用gem下载安装Rake

> gem install rake

安装成功后在自己Jekyll博客的根目录下创建一个没有后缀名的Rakefile文件,键入如下脚本：

```rake
require 'rake'
require 'yaml'

SOURCE = "."
CONFIG = {
  'posts' => File.join(SOURCE, "_posts"),
  'post_ext' => "md",
}

# Usage: rake post title="A Title"
desc "Begin a new post in #{CONFIG['posts']}"
task :post do
  abort("rake aborted: '#{CONFIG['posts']}' directory not found.") unless FileTest.directory?(CONFIG['posts'])
  title = ENV["title"] || "new-post"
  slug = title.downcase.strip.gsub(' ', '-')
  filename = File.join(CONFIG['posts'], "#{Time.now.strftime('%Y-%m-%d')}-#{slug}.#{CONFIG['post_ext']}")
  if File.exist?(filename)
    abort("rake aborted!") if ask("#{filename} already exists. Do you want to overwrite?", ['y', 'n']) == 'n'
  end

  puts "Creating new post: #{filename}"
  open(filename, 'w') do |post|
    post.puts "---"
    post.puts "title: \"#{title.gsub(/-/,' ')}\""
    post.puts "layout: post"


    post.puts "category: post"
    post.puts "tags: "
    post.puts "---"
  end
end # task :post
```
ok，脚本添加成功后，我们就可以使用rake来创建博客了。 
来一个测试，那就用 Hello World 来测试下。在Jekyll的根目录，打开终端后键入：

> rake post title="hello world"

如下图

[![rake](/static/post-image/rake.png)](/static/post-image/rake.png)

我是用Visual Studio Code来编辑博文的，通过Rakefile, 现在就可以很轻松的实现一键生成Jekyll文章模版了。

