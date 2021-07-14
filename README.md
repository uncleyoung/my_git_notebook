## Preface

Only take notes of very common usage.

## Local Usage

```shell
# 第一次使用会提示你填写邮箱和用户名，根据提示填写就可以了。没必要记下来了。
# ...

# Init a git. 进入到项目目录里，注意有个点
git init .

# 提交所有改变
git add . # or git add ./
git commit -m "my comments"

# 添加版本号（tag）
git tag -n # 列出所有tag，包括对应的comment
git tag new_tag_name # 添加
git tag -d tag_to_delete # 删除

# 回退（强制覆盖本地）
git reset --hard xxx # xxx是编号或者标签

# other
git log # 列出所有提交和注释
git diff # or git diff xxx # 列出区别
```

#### .gitignore 不起作用

一般是它只能忽略未跟踪的文件，所以你已经提交过，然后再加进来规则是不行的。

- 要么重新重头初始化git并提交，适用于刚刚建立的项目。

- 要么尝试将需要忽略的文件从版本追踪中删除，如下

  ```shell
  git rm --cached xxx git add xxx git commit -m "rm xxx to unversion it"
  ```

##### 一般用法

```shell
# this is comment
*.o # 通配符忽略某一类型文件
!*.cpp # 不要忽略
/filefolder/ # 忽略特定目录
filefolder/ # 忽略所有叫filefoler的目录
/filefolder/app.bin # 忽略某目录下某文件, or /filefolder/*.bin ?
.DS_Store # 忽略mac下特别烦人的隐藏文件
```



## Remote Usage

#### 一般流程

准备密匙

```shell
# 准备生成密匙（私、公）
ssh-keygen -t rsa -C "youremail@example.com" 

# 一直选yes或继续即可，最后一般在 ~/.ssh/id_rsa.pub 中存放公钥。（windows的git可能默认生成在/c/Users/xx/.ssh/xxx，生成指令会有打印提示）
# 将公钥（注意不包含最后的邮箱），设置到对应平台（github、gitee或公司内部服务器）的配置中即可。
# github、gitee等，一般都在setting中有ssh公钥添加的地方，随便找一找就能找到。

# 检查是否添加成功 ！注意提示输入时要输入yes，不要直接回车
ssh -T git@github.com
```

> !注意：验证公钥时，提示输入时要输入yes，不要直接回车。
>
> 如果不是私有项目，可以不用准备密匙。

后续操作：

```shell
# 查看
git remote # or git remote -v
# 添加
git remote add origin git@xxx.com:xxx/xxx.git
# 推送
git push -u origin main(or master) # 第一次
git push # 执行过-u以后
# 拉取
git pull 
git fetch
git merge
# 删除
git remote rm origin_name
# 切换, 直接修改(或者按照上面的，先删除在添加也可)
git remote set-url git@xxx.com:name/xxx.git
```

#### 提交tag

好像一般的push，到远程的不会包含本地已经打上的tag信息。

```shell
git push origin --tags # 提交所有的tag
git push origin some_specific_tag_name # 提交某一特定tag
```

#### 文件模式改变 100644 to 100755

当工作环境经常在windows/ubuntu/mac之间切换时，git如果严格比较，经常会发生文件mode改变的情况，例如下面这样

```shell
# 改变示例
uncledeMacBook-Pro-2:b9201_apps uncle$ git diff .gitignore 
diff --git a/.gitignore b/.gitignore
old mode 100644
new mode 100755
```

需要设置git不去比对文件mode

```shell
git config --add core.filemode false
```



#### Git嵌套问题

如果已经发生，先清理缓存，把嵌套的子目录解放出来。

```shell
# 先将嵌套的子git目录中的 .git 文件夹重命名或删除
# ... then,
git rm --cached sub_git_folder # 删除缓存
git add sub_git_folder # add back
git commit -m "your comment" # commit back
git push -u origin main(or master) # push again
```

提前准备

- 思路1：使用正统的submodule模式

- 思路2：邪道方法：准备一个脚本，每次提交使用如下脚本（假设linux系统）

  ```shell
  ## 经实践，这套方法虽然可以，但并不好用。
  ```

> ! 结论：尽量不要嵌套git，项目扁平组织，在本地分别clone。实在有必要需要母项目时，母项目可以事先用 gitignore 忽略子项目目录，然后在母项目中clone。如果一直有remote，也可以采用submodule模式。



#### 文件过大问题

很多git托管平台是限制上传的单个文件大小的，所以可能会发生这个问题。解决如下：

```shell
# 查询最大的10个文件
git rev-list --all | xargs -rL1 git ls-tree -r --long | sort -uk3 | sort -rnk4 | head -10
```

为了偷懒，写一个移除脚本 usage: `./xxx.sh filename_刚查到的`

```shell
#!/bin/sh
FILE=$1

# rm file
git log --pretty=oneline --branches -- $FILE
# rm history
git filter-branch --force --index-filter 'git rm --cached --ignore-unmatch --ignore-unmatch $FILE' --prune-empty --tag-name-filter cat -- --all
# clear local cache
rm -Rf .git/refs/original
rm -Rf .git/logs/
git gc
git prune
```

最后在强制提交

```shell
git push --force --all
```



#### Push的模式问题

github用push提交，首次可能会提示 matching 模式还是 simple 模式。自己看提示说明就好了。

matching是老模式，所有branch都提交。simple是新的，只提交当前branch。



#### 开源许可证书

[引用自阮一峰博客](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html )

![本地图片备份](./free_software_licenses.png)
