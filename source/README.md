# This is my blog
保存博客内容，由 blog 项目执行
```cmd
hexo clean && hexo deploy
```
推送至此，相当于单独将 public 仓库推送过来。

## 避免渲染此文档
在 blog `_config.yml` 设置：
```yml
# 指定 source 目录下哪些文件不被渲染
skip_render: README.md 
```
