# hbx-happy-blog

## 评论系统

博客文章页当前使用 `giscus`，配置统一放在 `hugo.yaml` 的 `params.comments`。

当前线上配置：

1. 仓库 `bx-h/hbx-happy-blog` 已开启 GitHub Discussions。
2. 已在仓库上安装 [giscus](https://github.com/apps/giscus) GitHub App。
3. 评论 discussion 会创建在 `Announcements` 分类下。

补充说明：

- 评论会按文章 `pathname` 自动映射到对应 Discussion。
- 单篇文章如果不想展示评论，可在 front matter 里加 `comments: false`。
