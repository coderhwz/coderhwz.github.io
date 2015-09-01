---
layout: post
title: homestead中添加自定义文件上传支持
keyword: homestead,laravel,scp,file upload
description: homestead中添加自定义文件上传支持
tags: 
    - homestead
---

项目有中存在自定义文件上传的需求,例如,老项目的nginx配置文件,所以添加homestead对文件上传的支持

修改 `scripts/homestead.rb` 添加以下代码

```ruby
    # 复制本机文件到虚拟机
    settings["files"].each do |file|
      t = "/tmp/" + File.basename( file["copy"] )
      config.vm.provision "file", source: file["copy"], destination: t
      config.vm.provision "shell" do |s|
        s.inline = "sudo mv " + t + " " + file["to"]
      end
    end
```
由于scp使用的vagrant用户,它对一些目录没有直接的操作权限, 所以先上传到临时目录,然后再转移到指定目录.

然后在初始文件`src/stubs/Homestead.yaml`中添加一两个密码的配置,例如

```yaml
files:
    - copy: ~/.gitconfig
      to: .gitconfig

```
