---
layout:     post
title:      "Arcanist 安装使用"
subtitle:   " \"适用于 macOS，git \""
date:       2018-05-09 12:00:00
author:     "Feng-xp"
header-img: "img/post-bg-2015.jpg"
catalog: true
tags:
    - Arcanist
    - iOS
    - Phabricator
---

> “Yeah It's on. ”

# 
### 安装
1. 安装 PHP5.2 及以后版本
2. 安装 Git
3. 安装 Arcanist
```bash
# 1. 安装
cd ~
mkdir -p Arcanist
cd Arcanist
git clone https://github.com/phacility/libphutil.git
git clone https://github.com/phacility/arcanist.git
# 2. 配置环境变量
vim ~/.bash_profile
# 添加一行
export PATH="$PATH:/Users/qzp/Arcanist/arcanist/bin/" #注意路径
# 生效
source ~/.bash_profile
# 验证
arc help
```
4. 更新 Arcanist
```bash
arc upgrade
```

### 项目配置
进入项目根目录, 创建 .arcconfig 文件,  编辑如下内容
```bash
# vim .arcconfig
{
	"project.name": "testProject", # 项目名称
	"phabricator.uri" : "http://cr.dev.pajkdc.com/", # phabricator 服务地址
	"editor": "vim", # 编辑器，推荐 vim
	"base": "git:HEAD^", # arc diff 基点，在下面命令使用中解释
	"arc.land.onto.default": "develop", # 要land到的分支，arc land 会用
	"arc.land.update.default": "merge", # 分支合并方式 merge/rebase
	"history.immutable": false # 是否使用 --squash merge ,具体参考文档
}
```
具体参考官方文档：[arcanist_new_project](https://secure.phabricator.com/book/phabricator/article/arcanist_new_project/)

### 其他配置
```bash
# 运行 arc get-config 获取所有的配置
arc get-config [key]
arc set-config [key] [value]
```

### 支持扩展配置
1. Lint
```bash
# arc linters
# vim .arclint
{
    "linters": {
        "merge-conflict": {
            "type": "merge-conflict"
        },
        "php-syntax": {
            "type": "php",
            "include": "(\\.php$)"
        }
    }
}
```
2. unit test

### 证书安装
在使用 arc 命令进行code review之前，需要安装证书，运行如下命令：
```bash
arc install-certificate
```
根据终端提示打开对应链接（phabricator 服务地址）获取 token，并粘贴到终端指定位置

### 简单code review流程
```bash
cd testProject
# 1.修改，提交代码
echo 'code review test' > README.md
git add .
git commit -m 'code review test'
# 2.发送代码改变到 Phabricator/Differential
arc diff # 如果运行 arc diff 之前没有commit代码，会提示先commit代码
# 3.进入 vim 编辑模式（根据.arcconfig文件配置），填写相关字段，保存退出，等待审核结果
# 4.审核通过
arc land
# 5.完成提交
```
![](2018-05-09-Arcanist%20%E5%AE%89%E8%A3%85%E4%BD%BF%E7%94%A8(macOS%EF%BC%8Cgit)/9E754E16-C7B0-4E55-BBE5-6CA0A2D50F27.png)

### 常用命令介绍
1. arc help：显示详细的命令帮助文档
2. arc diff：将多个 commits 创建成一个 diff（revision） ，发送到 Differential 审核
用法：arc diff
* 使用arc diff <commitA>：则 commits 范围就是  [commitA, HEAD]
* 使用arc diff，配置了 .arcconfig base 字段：则 commits 范围就是 [base, HEAD]
* 使用arc diff，未配置 base 字段：则 commits 范围就是 [git merge-base origin/currentBranch HEAD, HEAD] 也就是当前分支所有未提交的 commits
所以在 .arcconfig 中配置 "base": "git:HEAD^”，则表示只发送最新的一个 commit 去审核，但实际测试，最后 arc land 的时候会把没有经过审核的代码一起 push 上去，所以这个配置不建议使用
**常用参数**
--create : Always create a new revision.
--update revision_id：Always update a specific revision.
--edit：When updating a revision under git, edit revision information before updating.
注意：默认情况下是 create 还是 update 由 arc 自己判定，大部分情况都可以正确，如要严谨，请显示指定参数
3. arc land：发布通过 review 的 revision，通常作用的对象是 branch， tags
用法：arc land --onto target  --remote remote
target 取值：
				1. 由 —onto 参数 指定的值
				2. 当前 branch 的 upstream
				3. .arcconfig 配置中指定字段的值 arc.land.onto.default 
				4. ‘master’ in git / ‘default’ in Mercurial
remote 取值：
		1. 由—remote 参数指定的值
		2. 当前 branch 的 upstream
		3. origin in git
**常用参数**
--keep-branch：默认情况本地分支在land成功后会被删除，使用该参数来避免被删除
--merge：使用 --no-ff merge，而不是 --squash merge。. arcconfig 配置文件中 history.immutable 为 false 则默认带此参数
4. arc list：显示所有未关闭的 revision 以及状态
5. arc close-revision：关闭某个 revision
6. arc cover：查看某个提交的作者
7. arc patch：下载某个修订到本地进行审查
8. arc export：从 Differential 下载一个 patch
9. arc amend：提交 review 后更新 git commit message

### 参考
[arcanist_quick_start](https://secure.phabricator.com/book/phabricator/article/arcanist_quick_start/)
[arcanist_user_guides](https://secure.phabricator.com/book/phabricator/article/arcanist/)
[arcanist_diff](https://secure.phabricator.com/book/phabricator/article/arcanist_diff/)

#Phabricator#