# 常用命令

### 修改分支名

旧分支：`oldBranch`，新分支：`newBranch`

1. 将本地分支重命名
  ```git
  git branch -m oldBranch newBranch
  ```
2. 删除远程分支（没有则跳过此步骤）
  ```git
  git push --delete origin oldBranch
  ```
3. 将重命名后的分支推到远端
  ```git
  git push origin newBranch
  ```
4. 把修改后的本地分支与远程分支关联
  ```git
  git branch --set-upstream-to origin/newBranch
  ```

### rebase 合并分支

将分支`experiment`rebase（变基）到`master`。参考：Git官网[变基](https://git-scm.com/book/zh/v2/Git-%E5%88%86%E6%94%AF-%E5%8F%98%E5%9F%BA)

```git
git checkout experiment

git rebase master

git checkout master

git merge experiment
```

### 删除所有本地分支

```git
git branch | grep -v "master" | xargs git branch -D
```
