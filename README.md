## Preface

随笔记录最常见的用法，以及工作中遇到的小问题，以做参考和备忘。

## Local Usage

### 常见用法

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

##### commit 换行

```shell
git commit -m "1. 第一行 
2. 第二行" // 步骤二: 按Enter 输入第二行
```

### 忽略项目 .gitignore 

#### 一般用法

```shell
# this is comment
*.o # 通配符忽略某一类型文件
!*.cpp # 不要忽略
/filefolder/ # 忽略特定目录
filefolder/ # 忽略所有叫filefoler的目录
/filefolder/app.bin # 忽略某目录下某文件, or /filefolder/*.bin ?
.DS_Store # 忽略mac下特别烦人的隐藏文件
```

#### 不起作用

一般是它只能忽略未跟踪的文件，所以你已经提交过，然后再加进来规则是不行的。

- 要么重新重头初始化git并提交，适用于刚刚建立的项目。

- 要么尝试将需要忽略的文件从版本追踪中删除，如下

  ```shell
  git rm --cached xxx git add xxx git commit -m "rm xxx to unversion it"
  ```

#### 忽略所有，除了指定的（子）文件或文件夹

>换一种说法：只管理希望跟踪的项目。
>
>一般在特别大的项目，不希望、无法、没有必要提交整个内容；工作中只维护工作产出的情况。

方式一：尽量使用通配符，写法简单一些

```shell
# example of exclude all but some folder, and ignore items inside that folder
# 下面示意：只跟踪 /projects/common/apps 文件夹内所有内容，同时忽略其中的 .o .d 文件
*
!*/
!/projects/common/apps/** # 忽略指定的文件夹。** 可以忽略文件夹下所有，用一个*只忽略文件夹下文件
*.o # 再次排除不需要跟踪的东西
*.d
```

方式二：使用忽略对，逐步指定文件，写法繁琐一些

```shell
# example of another method (exclude end with "/" or not does not matter.)
# 下面示意：只跟踪 /projects/common/apps 文件夹内所有内容，同时忽略其中的 .o 文件
/*
!/projects
/projects/*
!/projects/common
/projects/common/*
!/projects/common/apps
*.o # 再次忽略不需要跟踪的项目
```

### 合并Merge

例如：目前的项目，基于一份庞大的SDK开发，我们的代码分布在这个SDK之中。无论是从效率还是可行性上考虑，我们无法用git远程托管整个SDK，而是用git托管我们的代码（或配置文件、脚本等）。当SDK发布了新版本后，我们需要进行在这种情况下的merge。

- 合并需要真的产生分支才可以合并，否则只是修改commit的指针。

- 需要注意commit的时间，很多规则以此为准。

方式一：通过remote，需要联网。

1. 将老SDK下的代码进行最后的commit-push，保持最新。
2. 准备好新的SDK，准备好.gitignore 和 配置好git config。add-commit 这个干净的sdk。
3. pull刚提交的branch-commit。手动合并所有冲突。>>>>>> ... ==== <<<<<

方式二：直接拷贝git文件，无需联网。

1. 先将老SDK和其代码进行最后的commit，保持干净。
2. 新建分支，并在此上尽量回退到最初版本，保证排除后续merge产生的复杂冲突情况。
3. 将老.git文件夹和.gitignore拷贝到新SDK下。直接add-commit，手动产生分支岔路。
4. 在临时分支上merge，同上。(也许需要在主分支上随便提交个东西，让主分支commit时间新于临时分支？)

> ! 如果SDK没有更新，简单的方式是直接拷贝.git+.gitignore，然后git reset --hard xxx 即可。

#### "无效"Merge举例

```shell
# 这样其实不算产生岔路，实际上还是在一条线上，无法“真的” merge (fast-forward)
mater:: *commit1 - *commit2 ------ *merge(commit5) # 实际就是commit3，并不是3和2的merge
temp ::                 \- *commit3 -/
# 这样才算是岔路
mater:: *commit1 - *commit2 - *commit6 - *merge(commit5)
temp ::                 \- *commit3 -----/
```

#### 冲突直接替换 --theirs or --ours

当有很多冲突，并且你明确知道可以使用哪个版本，一个个手动修文件内容要累死。可以直接覆盖指定使用哪个版本。

```shell
# 在冲突状态下
git checkout --theirs . # 或指定某文件（们）
git checkout --ours xxx.dts 
git add and commit xxx
```



#### 使用gitk查看分支路径

```shell
gitk --all # 也可以用 git log --graph --all 只是git更好看更丰富
```



## Remote Usage

### 一般流程

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



### 删除远程commit

```shell
# 1.通过找到想要退回到的哪个commit_id
git log
# 2.本地代码变成某个提交记录时刻的代码
git reset --hard commit_id
# 3.推送到服务器，一定要加 --force 参数 "master":对应的分支即可
git push origin HEAD:master --force
```



### 提交tag

好像一般的push，到远程的不会包含本地已经打上的tag信息。

```shell
git push origin --tags # 提交所有的tag
git push origin some_specific_tag_name # 提交某一特定tag
```



### 文件改动，模式改变 old mode 100755 new mode 100644, or vice versa

当工作环境经常在windows/ubuntu/mac之间切换时，可能会发生。

其中一个主要因素是windows的git不支持+x的文件模式，所以在windows下的git bash或GUI，可能会提示文件修改（模式改变），这时如果你继续commit，甚至push，就会连锁反应导致其它机器（linux/mac）拉取后又提示文件模式又被修改成了windows的模式。

如果想要保留文件权限，解决思路：

- !一定不要在windows端的git上commit或push！
- 在linux端，首先设回true（如果已经改过） `git config --local core.filemode true`
- 在linux端，reset 回最新的版本，用`git status`检查是否确实清理干净了。
- 在linux端，再将filemode设置为false `git config --local core.filemode false`
- 这时可以回到windows端，用`git status`检查一下是否可以了。
- ！注意，一定要清理后，filemode设为false之后，才能在windows端提交任何改动，这时git才会忽略文件模式的问题。
- 如果是反过来，以windows端某特有文件格式为主，也是一样的道理。

>optional: 或者涉及到+x的权限问题，全部在linux端操作，不用用windows的git。

> 另外：当上述上传之后，在拉取端，可以用mac拉取，默认的core.filemode是false的（或者是跟随上传的），这样直接可用。
>
> 如果用linux拉取，好像拉下来的core.filemode默认是true，还需要手动再改一下。

```shell
# 强迫症请注意，不要轻易使用--add，这会导致多出好几个一样的字段。
# git config --add core.filemode false 不要用这个，除非字段里没有
# 如果已经添加了，导致了多个重名字段，用 git config --local unset-all core.filemode 清除

# 其它git config用法，可以打help或者随便打个错误指令看下帮助说明就明白了。
```



### Git嵌套问题

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



### 文件过大问题

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



### Push的模式问题

github用push提交，首次可能会提示 matching 模式还是 simple 模式。自己看提示说明就好了。

matching是老模式，所有branch都提交。simple是新的，只提交当前branch。



### 开源许可证书

[引用自阮一峰博客](http://www.ruanyifeng.com/blog/2011/05/how_to_choose_free_software_licenses.html )

![本地图片备份](./free_software_licenses.png)
