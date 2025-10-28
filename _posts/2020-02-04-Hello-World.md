---
title: Hello World
date: 2020-02-04 12:12:12 +0800
categories: [生活, 闲杂]
description: 
pin: false
tags: [生活] 
---

## 前言

Blog 就这么开通了。

之前一直使用OneNote记录了大量笔记，后续因为OneNote一直不支持markdown格式，只能使用typora将md文件保存在本地。 

很早就想搭建自己的 Blog ，不曾料想踩了很多坑 ，遂放弃。。。

现在 Blog 总算是搭建好了。

---

## 正文

接下来说说搭建这个博客的技术细节。  

之前就有关注过 [GitHub Pages](https://pages.github.com/) + [Jekyll](https://jekyllrb.com/) 快速 Building Blog 的技术方案，非常轻松时尚。

其优点非常明显：

* **Markdown** 带来的优雅写作体验
* Git workflow ，**Git Commit 即 Blog Post**，非常方便
* 利用 GitHub Pages 的域名和免费无限空间，不用自己折腾主机
	* 如果需要自定义域名，也只需要简单改改 DNS 加个 CNAME 就好了 
* Jekyll 的自定制非常容易，基本就是个模版引擎

---

主题我直接 套用了[jekyll](https://github.com/cotes2020/jekyll-theme-chirpy) 的模板，简单粗暴，不过遇到了很多坑😂，好在都填完了。。。

根据 [Chirpy 的官方文档]([Getting Started | Chirpy](https://chirpy.cotes.page/posts/getting-started/))，需要先在 github 中对 [https://github.com/cotes2020/chirpy-starter](https://github.com/cotes2020/chirpy-starter) 选择 `Use this template` 来创建一个新仓库，并将这个仓库命名为 `githubusername.github.io` 。



## 创建站点存储库

创建站点存储库时，有两个选项：

### 方法1. 使用启动器 chirpy-starte（推荐）

这种方法简化了升级，隔离了不必要的文件，非常适合想要专注于以最少配置编写的用户。

1. 登录到 GitHub 并导航到[**启动器 (chirpy-starte)**](https://github.com/cotes2020/chirpy-starter)
2. 选择 `Use this template` --> `Creat a new repository`, 创建一个新存储库
3. 将新存储库命名为 `<username>.github.io` （`username`替换为自己的小写 GitHub 用户名)

### 方法2. Fork存储库

这种方法方便修改功能或 UI 设计，但在升级过程中会带来挑战。因此，除非熟悉 Jekyll 并计划大量修改此主题，否则不要尝试此作。

1. 登录到 GitHub。
2. [Fork存储库](https://github.com/cotes2020/jekyll-theme-chirpy/fork)。
3. 将新存储库命名为 `<username>.github.io` （`username`替换为自己的小写 GitHub 用户名)

## 修改 config 进行个性化

此时，我已经拥有了一个自己的在线 git仓库，将这个仓库 clone 到本地。

打开 clone 下来的目录，找到名为 `_config.yml` 的文件，进行自定义设置。

每个主题的 config 都会大同小异，最关键的一项是 `url` ，填入仓库名称，也就是 `username.github.io` 。其他的按照文件上的说明填写即可。

需要特别提醒评论功能的配置，有 disqus, utterances, giscus 三种方式可选。选择的方式不同，配置的方式也会不同。

我选择的是 giscus ，进入 [giscus](https://giscus.app/zh-CN)，填写仓库等信息，并按要求进行一系列简单的配置。结束后中页面底端会获取到 repo, repo_id, category, category_id 等信息，填入  `_config.yml`。

```yaml
comments:
  # Global switch for the post-comment system. Keeping it empty means disabled.
  provider: giscus  # [disqus | utterances | giscus]
  # The provider options are as follows:
  disqus:
    shortname: # fill with the Disqus shortname. › https://help.disqus.com/en/articles/1717111-what-s-a-shortname
  # utterances settings › https://utteranc.es/
  utterances:
    repo: # <gh-username>/<repo>
    issue_term: # < url | pathname | title | ...>
  # Giscus options › https://giscus.app
  giscus:
    repo: # <gh-username>/<repo>
    repo_id: ''
    category: ''
    category_id: ''
    mapping: 'pathname' # optional, default to 'pathname'
    strict: # optional, default to '0'
    input_position: 'top' # optional 'top/bottom', default to 'bottom'
    lang: zh-CN # optional, default to the value of `site.lang`
    reactions_enabled: '1' # optional, default to the value of `1`
```



## 部署站点

> [!Tip]
>
> 部署前需检查 `_config.yml` 文件配置正确（Jekyll 的核心配置文件）

重点确认两个关键参数：

- `url`：通常是网站的根域名，如 GitHub Pages 的 `https://[username].github.io`

- `baseurl`: 仅在将您的网站托管在子目录中时才需要

  仅在两种场景下需要设置：

  - 场景 1：[**GitHub Pages**](https://pages.github.com/) 上托管的[项目站点](https://docs.github.com/en/pages/getting-started-with-github-pages/about-github-pages#types-of-github-pages-sites) ，且不使用自定义域名；

  - 场景 2：网站部署在非 GitHub Pages 的服务器上，且需要通过 “baseurl” 访问，如: `https://your-server.com/project-name`

     设置格式需以斜杠开头，示例为 `/project-name` (替换为实际项目名）。若无需 baseurl，此参数可留空或注释。



现在，可以使用以下方法来部署自己的 Jekyll 站点

### 方法1. 使用 Github Actions 进行部署 (推荐)

1）进入刚才创建的仓库 `settings` 页面，在左侧导航栏选择 `pages`。

2）找到 `Build and deployment`，将 `source` 选择为 `GitHub Actions`。

3）回到本地仓库，将任何提交推送到 GitHub 以触发`Actions`工作流。

在存储库的 `Actions` 选项卡中，就可以看到`Build and Deploy`工作流正在运行。构建完成并成功后，将自动部署站点。

等待几分钟，Deployments 完成后就可以通过  GitHub 提供的 URL 进入自己的博客站点啦！

### 方法2. 部署在自建服务器

对于自建服务器，需要在本地计算机上构建站点，然后将站点文件上传到服务器。

导航到源项目的根目录，然后使用以下命令构建站点：

```bash
$ JEKYLL_ENV=production bundle exec jekyll b
```

默认情况下生成的站点文件放置在源项目的根目录中

## 写下第一篇文章

在 *_posts 文件夹内新建一个 markdown 文件，命名格式为`YYYY-MM-DD-title.md`，例如 `2020-10-10-Hello World.md`，输入以下内容。

```markdown
---
title: hello world
date: 2020-10-10 21:00:00 +0800
categories: [TOP_CATEGORY, SUB_CATEGORY]
tags: []
pin: false
---

hello world!
```

上面使用 `---` 隔开的内容是这份文件的元数据，需要包含标题、日期、类别、标签等信息。

帖子的**布局**已默认设置为`post`，因此无需在 Front Matter 块中添加**layout**变量。

## 升级站点

参考：[Upgrade Guide · cotes2020/jekyll-theme-chirpy Wiki](https://github.com/cotes2020/jekyll-theme-chirpy/wiki/Upgrade-Guide)

