---
title: "Git Commit 规范化"
date: 2021-08-18T12:26:50+08:00
draft: true
toc: true
images:
tags: 
  - engineering
---

### Conventoional 规范

```
<type>(<scope>): <subject>
<空行>
<body>
<空行>
<footer>
```

- type 提交类型 
  - feat  新功能
  - build 对构建系统或外部依赖项进行了修改
  - ci 对 CI 配置文件或脚本进行了修改
  - perf  提供性能的代码更改、优化
  - fix     改bug
  - docs 文档
  - style 格式
  - refactor  重构
  - test 测试
  - chore 构建过程或辅助工具的变动
  - sync  同步主线或分支的bug
  - merge 代码合并
  - revert 回滚到上一个版本
- [scope] commit  影响范围(模块、层)、特性
-  subject  简短描述
  - 结果不加句号或其他标点符号
  - 命令式，现在时态
  - 中文/不要大写字母
- [body] 详细描述  \n 换行
- [footer] 结尾，不兼容说明、关闭 issue

```
Footer
BREAKING CHANGE: xxx
	Before:
		xxx
	After：
		xxx
		
Close $issueID		
```

- revert

  当前 commit 用于撤销以前的 commit ，必须以 revert: 开头，后面跟着被撤销的 commit 的 Header
  Body 部分格式固定，必须写成 This reverts commit 被撤销commit SHA 标识符. 
  如果当前 commit 与被撤销的 commit ,在同一个发布(release)里面，他们都不会出现在 change log 里面，如果两者在不同的发布，当前的 commit ,会出现在 Change log 的 Reverts 小标题里面

  ```
  revert:<type>(<scope>): <subject>
  <BLANK LINE>
  This reverts commit hash
  <other-body>
  <BLANK LINE>
  <footer>
  ```

  

### Commitizen 交互

```
#npm 淘宝源
npm config set registry https://registry.npm.taobao.org 
# 安装 commit message cli 交互式工具
npm install -g commitizen
#Adapter 配置适配器实现 conventional commit 规范
npm install -g cz-conventional-changelog
#全局配置
echo '{ "path": "cz-conventional-changelog" }' > ~/.czrc
git cz
```

### Use Cli

``git cz / git-cz 交互式提交``

### CommitLint Husky 消息校验

[commitlint ](https://commitlint.js.org/#/)

```
全局安装
npm install -g @commitlint/cli @commitlint/config-conventional
使用conventional规范
echo "module.exports = {extends: ['@commitlint/config-conventional']};" > commitlint.config.js
测试
echo 'test: test commitlint' | commitlint

每次 commit 之后检测
commitlint -x '@commitlint/config-conventional' -e
 -e / --edit 读取 commit 最后一条记录
检测最近几条记录
commitlint -x '@commitlint/config-conventional' -f HEAD~1 
-t/--to 检测区间段
```

### changelog

```
安装
npm install -g conventional-changelog-cli
生成 changelog
conventional-changelog -p angular
 -r 0 代表将 commit 记录从头至尾全部生成 changelog
```

### Conventional-changelog

```
安装
npm i -g standard-version
执行
standard-version
```

- 更新 CHANGELOG.md
- commit
- 新版本 tag

参数

- --dry-run 打印一下要做的，不真正执行
- --first-release / -f 首次发布，不会升级版本号，第一次需要
- --commit-all / -a 
- --release-as  / -r 手工指定下一个版本号，默认 [major，minor，patch]
- --prerelease / -p 预发布版本，默认产生的 tag  eg: v1.0.1-0 ，跟上可选的 tag id , eg -p alpha，最终 tag: v1.0.1-alpha.0

