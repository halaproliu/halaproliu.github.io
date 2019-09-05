
# 使用shell脚本获取当前git分支并提交远程分支

```bash
#!/bin/bash
hasGit=`which git` # 判断是否存在git
msg=${1:-'auto commit'} # 获取终端输入的第一个参数，若为空则为auto commit
if [ ! $hasGit ];then
  echo 'Please download git first!';
  exit 1;
else 
  result=`git branch | grep "*"` # 获取分支名
  curBranch=${result:2} # 去除多余的*
  git add .
  git commit -m "$msg"
  git push github $curBranch # 提交代码到github(修改了远程项目名)
  git push gitee $curBranch # 提交代码到gitee
fi
```
