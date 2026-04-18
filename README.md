# hbx-happy-blog

## 评论系统

博客文章页已接入 `utterances` 评论组件，配置位于 `hugo.yaml` 的 `params.comments`。

启用前请确认：

1. 仓库 `bx-h/hbx-happy-blog` 已开启 GitHub Issues。
2. 已在仓库上安装 [utterances](https://utteranc.es/) GitHub App。

补充说明：

- 评论会按文章 `pathname` 自动映射到对应 Issue。
- 单篇文章如果不想展示评论，可在 front matter 里加 `comments: false`。
