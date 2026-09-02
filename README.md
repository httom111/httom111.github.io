# HTTOM39 Blog

个人博客源码，基于 [Astro](https://astro.build) 和 [Firefly](https://github.com/CuteLeaf/Firefly) 主题模板（Firefly 基于 [Fuwari](https://github.com/saicaca/fuwari) 二次开发）。

## 开发

```bash
pnpm install
pnpm run dev
```

## 写新文章

在 `src/content/posts/` 下新建一个 `.md` 文件，参考已有文章的 frontmatter 格式即可。图片放在同目录的 `images/` 子文件夹下，用相对路径 `./images/xxx.jpg` 引用。

## 部署

push 到 `main` 分支后，GitHub Actions 会自动构建并发布到 GitHub Pages，无需手动操作。

## 主题配置

站点相关配置集中在 `src/config/` 目录下，各文件均有中文注释说明。
