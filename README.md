## Preface

Only take notes of very common usage.



## Local Usage

```shell
# 第一次使用会提示你填写邮箱和用户名，根据提示填写就可以了。没必要记下来了。
# ...

# 提交所有改变
git add . # or git add ./
git commit -m "my comments"

# 添加版本号（tag）
git tag -n # 列出所有tag，包括对应的comment
git tag new_tag_name # 添加
git tag -d tag_to_delete # 删除

# other
git log # 列出所有提交和注释
git diff # or git diff xxx # 列出区别
```



## Remote Usage

#### 嵌套git问题

如果已经发生，先清理缓存，把嵌套的子目录解放出来。

```shell
# 先将嵌套的子git目录中的 .git 文件夹重命名或删除
# ... then,
git rm --cached sub_git_folder # 删除缓存
git add sub_git_folder # add back
git commit -m "your comment" # commit back
git push -u origin main(or master) # push again
```

##### 提前准备

- 思路1：使用正统的submodule模式

- 思路2：邪道方法：准备一个脚本，每次提交使用如下脚本（假设linux系统）

  ```shell
  ```

  

#### 开源许可证书

[引用自阮一峰博客]: http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html	"block"





#### matching or simple mode, when push.

github用push提交，首次可能会提示 matching 模式还是 simple 模式。自己看提示说明就好了。

matching是老模式，所有branch都提交。simple是新的，只提交当前branch。

一般流程



文件过大问题

```shell
# 查询最大的10个文件
git rev-list --all | xargs -rL1 git ls-tree -r --long | sort -uk3 | sort -rnk4 | head -10
```



切换远程地址



版权

