# Sub2API 项目 Git 维护与开发交接文档

## 当前项目结构（已完成）

当前项目已经从“直接修改线上源码”转换为“正规 Git Fork Workflow”。

现在结构如下：

```text
官方仓库（upstream）
Wei-Shaw/sub2api
        ↑ upstream

你的 Fork（origin）
Lucaschua88/sub2api
        ↑ origin

你的开发分支
secretapi-custom
```

---

# 当前服务器状态

项目目录：

```bash
/www/wwwroot/sub2api
```

当前开发 branch：

```bash
git branch
```

应该看到：

```text
* secretapi-custom
```

---

# 非常重要的规则（必须遵守）

# 1. 永远不要在 main 开发

禁止：

```bash
git checkout main
# 然后直接修改代码
```

原因：

main 是用于同步官方更新。

所有自定义开发：

* chat 功能
* branding
* SecretAPI
* frontend 修改
* auth 修改
* payment 修改

全部必须在：

```text
secretapi-custom
```

branch 进行。

---

# 2. 每次开发前先确认 branch

开发前执行：

```bash
git branch
```

确认：

```text
* secretapi-custom
```

如果不是：

```bash
git checkout secretapi-custom
```

---

# 3. 不要直接修改线上代码但不 commit

错误做法：

```bash
nano xxx.go
# 改完直接运行
```

这样：

* 无法 rollback
* 更新会爆炸
* 不知道改了什么

正确做法：

```bash
git status
git add .
git commit -m "描述修改"
git push
```

---

# 日常开发流程（标准）

## 修改代码

修改后：

```bash
git status
```

查看改动。

---

## 提交代码

```bash
git add .
```

然后：

```bash
git commit -m "add chat feature"
```

commit message 建议：

* add xxx
* fix xxx
* improve xxx
* refactor xxx

不要写：

* update
* test
* 123
* ok

---

## 上传到 GitHub

```bash
git push
```

因为已经绑定 tracking branch：

```text
origin/secretapi-custom
```

以后直接 push 即可。

---

# 如何查看当前修改

```bash
git status
```

查看详细差异：

```bash
git diff
```

查看修改文件：

```bash
git diff --name-only
```

---

# 如何查看 commit 历史

```bash
git log --oneline --graph --decorate --all -20
```

---

# 官方更新同步流程（非常重要）

## 1. 切回 main

```bash
git checkout main
```

---

## 2. 获取官方更新

```bash
git pull upstream main
```

这里：

```text
upstream = 官方 repo
```

---

## 3. 回到 custom branch

```bash
git checkout secretapi-custom
```

---

## 4. 合并官方更新

```bash
git merge main
```

如果出现 conflict：

* 不要 panic
* Git 会明确提示哪些文件冲突
* 修复后 commit 即可

---

# 更新前必须做的事情

更新官方前：

```bash
git status
```

必须确认：

```text
working tree clean
```

否则：

* merge 容易爆炸
* 修改可能丢失

---

# 非常危险的命令（不要乱用）

## 危险：

```bash
git reset --hard
```

会直接删除未提交修改。

---

## 危险：

```bash
git clean -fd
```

会删除未追踪文件。

---

## 危险：

```bash
git pull
```

如果不清楚当前 branch 与 remote 状态。

---

## 危险：

```bash
git checkout main
# 然后开发
```

会污染 main。

---

# 如何备份（推荐）

即使已经 push GitHub，重要更新前仍建议备份：

```bash
cd /www/wwwroot
cp -a sub2api sub2api-backup
```

或者：

```bash
tar -czvf sub2api-backup.tar.gz sub2api
```

---

# 当前已经确认的 custom 功能

当前 custom branch 已包含：

* SecretAPI branding
* Chat 页面
* ChatGPT 风格 UI
* Chat sidebar navigation
* Layout 修改
* Frontend 修改
* Docker 修改

相关 commit：

```text
680758b3 Fix duplicate AppLayout import
9441bcdf Use app layout for chat page
d0dc75a7 Add chat sidebar navigation
7783d8e3 Improve ChatGPT style chat page
c636ba9f Customize SecretAPI branding and frontend changes
```

---

# GitHub 结构

你的 Fork：

[https://github.com/Lucaschua88/sub2api](https://github.com/Lucaschua88/sub2api)

官方仓库：

[https://github.com/Wei-Shaw/sub2api](https://github.com/Wei-Shaw/sub2api)

---

# Token 注意事项

当前服务器使用 GitHub Personal Access Token。

注意：

* 不要公开 token
* 不要截图 token
* 不要 commit token
* 不要放进前端代码

以后如果 token 泄露：

GitHub：

```text
Settings
→ Developer Settings
→ Personal Access Tokens
→ Delete token
```

重新生成即可。

---

# 推荐未来优化（可后续做）

后续可以慢慢增加：

* SSH Deploy Key
* GitHub Actions 自动部署
* Dev / Prod 分支
* Docker CI/CD
* 自动备份
* 自动 build
* staging server

但目前核心 Git 结构已经正确。

---

# 当前项目状态结论

当前项目已经从：

```text
直接修改线上源码
```

升级为：

```text
正规 Git Fork 开发结构
```

目前：

✅ 官方历史保留
✅ custom branch 独立
✅ chat 功能已 commit
✅ GitHub 已备份
✅ upstream 已建立
✅ 可安全更新官方
✅ 可长期维护
