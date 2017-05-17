# Git操作

```shell
1. 批量删除本地分支：
git branch -D 3.2 3.2.1 3.2.2
git branch -D 3.2.*
git branch -D `git branch | grep -E '^3\.2\..*'`
```

