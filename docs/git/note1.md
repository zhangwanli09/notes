# 修改分支名

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
