---
title: Hexo 博客添加 sitemap
date: 2019-11-22 19:20:20
categories:
- hexo
tags:
- sitemap
---
为博客添加 sitemap 并将博客添加至搜索引擎

## Google Search Console

到博客根目录下运行 command 安装插件

```bash
npm install hexo-generator-sitemap --save
```

在根目录下的 _config.yml 中添加 sitemap 配置

```yml
# Config sitemap to enable SEO
sitemap:
  path: sitemap.xml
```

重新生成文件，启动 server

```bash
hexo g
hexo s
```

访问 url/sitemap.xml 可以看到新生成的 sitemap xml 文件

访问 [Google Search Console](https://search.google.com/search-console/about) 注册你的页面。
如果你的博客挂在 github 上，选右边的那个，输入你的 github 博客地址，比如我的是 `https://jack-zheng.github.io/hexo`
![资源类型](seo01.jpg)

结下来是验证所有权, 选择 HTML 标记会简单一点。点击他，你会得到一串码。复制它然后到 themes/next/_config.yml 中，找到 `google_site_verification:` 将值写在后面

![所有权验证](seo02.png)

添加完后，将你的博客部署，在次访问是查看页面源码，你会发现头部多了一段 meta 数据

![文件头](seo_meta.png)

然后点击 Verify 按钮，验证成功

![所有权验证](seo03.png)

点击 `站点地图` 在 1 处填写你的 sitemap 地址。添加完成后，他会显示在 2 处

![添加站点](seo04.png)

等一段时间后，google 就会将你的博客抓去出来了， 通过在搜索框中输入 `site:https://jack-zheng.github.io/` 可以看到结果，我是在第二天看的，不是很清楚精确需要等多久

![Google Search 结果](seo05.png)

## 博客添加图片引用

* [官方文档](https://hexo.io/zh-cn/docs/asset-folders.html)

由于我的博客是挂载在 subdirectory 下面的，在 source 下面创建 images 的方案不生效，也没有找到对应的解决办法，我还以为可以在 _config.yml 里面配置来着，残念 ┑(￣Д ￣)┍

最后采用了 post_asset_folder 的配置，这个配置默认就有的，默认关闭，把他设置成 true 打开，之后每次创建新 post 的时候，会在 _posts 下面新建文件夹，将你要上传的图片放在里面，在 post 正文中使用 `![Google Search 结果](seo05.png)` 引用即可

```bash
# folder structure sample
source
├── _posts
│   ├── git-commands.md
│   ├── hexo-comments.md
│   ├── hexo-search-seo
│   │   ├── seo01.jpg
│   │   ├── ...
│   │   └── seo05.png
│   ├── hexo-search-seo.md
│   ├── ...
│   └── setup-hexo-tag-category.md
├── categories
│   └── index.md
└── tags
    └── index.md
```
