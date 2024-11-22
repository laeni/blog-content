---
title: GIT 笔记
tags: GIT
author: Laeni
date: 2024-10-29
updated: 2024-10-29
---

# 脚本

## 批量将标签加上前缀'v'

```sh
# 遍历每个标签
for tag in $(git tag); do
  if [[ $tag != v* ]]; then
    # 创建新的带有前缀 v 的标签并将新标签推送到远程
    git tag "v$tag" "$tag"
    git push origin tag "v$tag"
    # 删除标签
    git tag -d "$tag"
    git push origin --delete "$tag"
  fi
done
```

