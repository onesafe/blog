---
title: Hexo集成Gitalk评论
date: 2018-07-07 23:57:38
category: 技术
tags: [技术]
---

### 开端

想在自己的博客系统里集成评论系统，到网上一搜。最后发现Gitalk评论系统用的人最多，受称赞比较多


Gitalk Demo: https://gitalk.github.io/


### 注册GitHub Application
在GitHub上注册新应用，链接：https://github.com/settings/applications/new

参数说明：
* Application name:  # 应用名称，我的名称是hexo-nsxt-comments
* Homepage URL:  # 网站URL，如https://branw.cn
* Application description:    # 描述，随意
* Authorization callback URL:   # 网站URL，https://branw.cn


`填写网站URL的时候，注意要填写你自己域名的网址，没有的话就写github.io网址，我在这个地方花了好多时间，最后写对了网址URL就成功了`

注册完成后会得到ClientID, Client Secret，后面配置_config.yml的时候会用到


### 创建repo
创建一个与应用名称一样的repo，用来存放评论
例如：我的repo名字就是：hexo-nsxt-comments


### 配置Gitalk
新建/layout/_third-party/comments/gitalk.swig文件，并添加内容：
```jinja2
{% if not (theme.duoshuo and theme.duoshuo.shortname) and not theme.duoshuo_shortname %}
  {% if theme.gitalk.enable %}
    {% if page.comments %}
      <script src="https://unpkg.com/gitalk/dist/gitalk.min.js"></script>
      <script src="https://cdn.bootcss.com/blueimp-md5/2.10.0/js/md5.js"></script>
      <script type="text/javascript">
        const gitalk = new Gitalk({
          clientID: '{{theme.gitalk.clientID}}',
          clientSecret: '{{theme.gitalk.clientSecret}}',
          repo: '{{theme.gitalk.repo}}',
          owner: '{{theme.gitalk.owner}}',
          admin: '{{theme.gitalk.admin}}'.split(','),
          pagerDirection: '{{theme.gitalk.pagerDirection}}',
          id: md5(window.location.pathname),
          distractionFreeMode: false
        })
        gitalk.render('gitalk-container')
      </script>
    {% endif %}
  {% endif %}
{% endif %}
```

修改/layout/_partials/comments.swig
```jinja2
{% elseif theme.gitalk.enable %}
  <div id="gitalk-container"></div>
  <link rel="stylesheet" href="https://unpkg.com/gitalk/dist/gitalk.css">
```

修改layout/_third-party/comments/index.swig，在最后一行添加内容：
```jinja2
{% include 'gitalk.swig' %}
```

在主题配置文件next/_config.yml中添加如下内容：
```yaml
gitalk:
  enable: true
  owner: onesafe 
  repo: hexo-nsxt-comments  
  clientID: xxxxxxxxx
  clientSecret: xxxxxxxxx
  admin: onesafe 
  pagerDirection: last
```


### 授权
最后一步了
这时候打开博客，发现评论是空的，没有创建ISSUE
如果我们前面的都配置成功的话，尤其是Authorization callback URL配置成功的话
这时候我们只需要用admin账户登录GitHub授权就大功告成了

`下面的图就是授权的过程，点击授权就OK了`
![](https://wx2.sinaimg.cn/mw690/006yibQ1ly1ft0i7bm84kj30qo0xl77i.jpg)