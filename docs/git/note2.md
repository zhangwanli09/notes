# rebase 合并分支

将分支`experiment`rebase（变基）到`master`。

```git
git checkout experiment

git rebase master

git checkout master

git merge experiment
```

> 参考：
>
> Git官网[变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)
