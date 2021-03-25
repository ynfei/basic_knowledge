# git tag 快速使用

- 列出所有的tag

  git tag 

- 创建tag

  git tag -a v1.0.0 -m "release 1.0.0"

- 将tag推送到远程

  git push origin v1.0.0

- 删除本地tag

  git tag -d v1.0.0

- 删除远程tag

  git push origin :refs/tags/v1.0.0