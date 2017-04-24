---
title: Hello World
date: 2017-04-14 10:12:09
---
Welcome to [Hexo](https://hexo.io/)! This is your very first post. Check [documentation](https://hexo.io/docs/) for more info. If you get any problems when using Hexo, you can find the answer in [troubleshooting](https://hexo.io/docs/troubleshooting.html) or you can ask me on [GitHub](https://github.com/hexojs/hexo/issues).<!-- more -->

## Quick Start
---
### Init
```bash
$ hexo init
```
More info: [Setup](https://hexo.io/docs/setup.html)

### Create a new page
```bash
$ hexo new page about
```
More info: [Page](http://theme-next.iissnan.com/theme-settings.html)

### Create a new post

``` bash
$ hexo new "My New Post"
```

More info: [Writing](https://hexo.io/docs/writing.html)

### Run server

``` bash
$ hexo server
```

More info: [Server](https://hexo.io/docs/server.html)

### Generate static files

``` bash
$ hexo generate
```

More info: [Generating](https://hexo.io/docs/generating.html)

### Deploy to remote sites

``` bash
$ hexo deploy
```

More info: [Deployment](https://hexo.io/docs/deployment.html)

## Hexo Theme
---

### Installation
1. Get it from GitHub
```bash
 $ git clone https://github.com/iissnan/hexo-theme-next themes/next
```
2. Add it to `_config.yml`
```bash
 theme: next
```
### Tags Page
>Add a tags page contains all tags in your site.

1. Create a page name `tags`
```bash
hexo new page "tags"
```

2. Edit tags page, set page type to `tags`
```yml
title: All tags
date: 2017-04-24 16:01:05
type: tags
- Swift
- OC
- Java
```
3. Add `tags` to theme `_config.yml`
```yml
menu:
  home: /
  archives: /archives
  tags: /tags
```

### Categories page
> Add a categories page contains all categories in your site

1. Create a page named `categories`
```bash
hexo new page "categories"
```
2. Edit categories page, set page type to `categories`
```yml
title: All categories
date: 2017-04-24 16:02:15
type: "categories"
- Swift
- Android
- Other
```
3. Add categories to theme `_config.yml`
```yml
menu:
  home: /
  archives: /archives
  categories: /categories
```

### Social Media
> NexT can automatically add links to your Social Media accounts:

```yml
social:
  GitHub: your-github-url
  Twitter: your-twitter-url
  Weibo: your-weibo-url
  DouBan: your-douban-url
  ZhiHu: your-zhihu-url
```

More info: [NexT](https://github.com/iissnan/hexo-theme-next)
