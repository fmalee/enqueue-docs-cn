# Enqueue中文文档

一直在用 Symfony 自带的 Messenger 做异步和队列。突然发现Enqueue改变挺大的，所以兴起尝试一下。

然后顺手翻译一下中文文档。虽然英文文档也挺明了简单，但是总不如英文看得亲切，速查时脑子也反应更迅速。

## 文档目录

- [Enqueue中文文档（Github Pages）](https://fmalee.github.io/enqueue-docs-cn/concepts/)

## 同步记录

- `master`
  - [3731d696da04475e0ee4944a932b06ddd50a4c63](https://github.com/php-enqueue/enqueue-dev/commit/3731d696da04475e0ee4944a932b06ddd50a4c63)（2021-01-22）

## 未译章节

目前所有章节已翻译完成

## 术语约定

有不少专用的队列术语，可以到文档的[关键概念](https://fmalee.github.io/enqueue-docs-cn/concepts/)中查看，这里就不一一列举。

## GitHub Pages

- 将本文档上传到自己的GitHub仓库
- 依次进入仓库设置（Settings）-> `Pages`。
- `Source` 中选择分支、目录（/docs），然后保存。
- 过一会儿就可以在上面的界面中看到新生成的文档地址。

## 本地测试

要在本地运行此文档，您可以在本地计算机上创建 Jekyll 环境或使用 docker 容器。
要运行 docker 容器，您可以使用来自仓库根目录的命令：

```shell
docker run -p 4000:4000 --rm --volume="${PWD}/docs:/srv/jekyll" -it jekyll/jekyll jekyll serve --watch
```

一旦构建完成，文档将可从 http://localhost:4000/ 访问，并在更改时自动重建。

## 资源

- [官方Github](https://github.com/php-enqueue/enqueue-dev/tree/master/docs)
- [官方文档](https://php-enqueue.github.io/)

