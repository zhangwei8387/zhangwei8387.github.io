# 贡献指南

感谢你对本项目的关注！本文档将指导你如何向本项目提交 Pull Request (PR) 并将其合并到 master 分支。

## 目录

- [开发流程](#开发流程)
- [如何提交PR](#如何提交pr)
- [PR审查标准](#pr审查标准)
- [合并PR到master](#合并pr到master)
- [常见问题](#常见问题)

## 开发流程

### 1. Fork 和 Clone 仓库

首先，Fork 本仓库到你的 GitHub 账号下，然后克隆到本地：

```bash
git clone https://github.com/你的用户名/zhangwei8387.github.io.git
cd zhangwei8387.github.io
```

### 2. 添加上游仓库

```bash
git remote add upstream https://github.com/zhangwei8387/zhangwei8387.github.io.git
```

### 3. 创建功能分支

从 master 分支创建一个新的功能分支：

```bash
git checkout master
git pull upstream master
git checkout -b feature/你的功能名称
```

### 4. 本地开发和测试

安装依赖：

```bash
npm install
```

本地预览：

```bash
npm run server
```

构建网站：

```bash
npm run build
```

清理构建产物：

```bash
npm run clean
```

## 如何提交PR

### 1. 提交你的更改

```bash
git add .
git commit -m "描述你的更改"
```

提交信息应该清晰地描述你做了什么更改。

### 2. 推送到你的 Fork

```bash
git push origin feature/你的功能名称
```

### 3. 创建 Pull Request

1. 访问你 Fork 的仓库页面
2. 点击 "New Pull Request" 按钮
3. 选择 base repository 为 `zhangwei8387/zhangwei8387.github.io`，base 分支为 `master`
4. 选择 compare repository 为你的 Fork，compare 分支为你的功能分支
5. 填写 PR 标题和描述：
   - 标题应简洁明了
   - 描述中说明你做了什么更改、为什么要做这个更改
   - 如果解决了某个 Issue，在描述中引用它（例如：`Fixes #123`）
6. 点击 "Create Pull Request"

### 4. 等待自动检查

创建 PR 后，GitHub Actions 会自动运行以下检查：

- ✅ 代码能否成功构建
- ✅ 构建产物是否正确生成
- ✅ 依赖安装是否成功

你可以在 PR 页面的 "Checks" 标签下查看检查结果。如果检查失败，请根据错误信息修复问题并推送新的提交。

## PR审查标准

提交的 PR 应该满足以下标准：

- [ ] 代码能够成功构建（`npm run build` 通过）
- [ ] 如果修改了内容，本地预览正常（`npm run server`）
- [ ] 提交信息清晰明了
- [ ] PR 描述详细说明了更改内容
- [ ] 没有引入不必要的文件（如 `node_modules`、构建产物等）
- [ ] 遵循项目的现有代码风格和结构

## 合并PR到master

### 方式一：通过 GitHub 网页界面合并（推荐）

这是最简单也是推荐的方式：

1. **等待自动检查通过**：确保 PR 页面显示所有检查都通过（绿色对号 ✅）
2. **代码审查**：如果有其他贡献者，等待他们的审查和批准
3. **点击合并按钮**：
   - 在 PR 页面底部，点击 "Merge pull request" 按钮
   - 选择合并方式（推荐使用默认的 "Create a merge commit"）
   - 点击 "Confirm merge"
4. **删除分支**（可选）：合并后可以删除功能分支以保持仓库整洁

### 方式二：通过命令行合并

如果你有仓库的写入权限，也可以通过命令行合并：

```bash
# 1. 切换到 master 分支
git checkout master

# 2. 更新 master 分支到最新
git pull origin master

# 3. 合并功能分支
git merge --no-ff feature/你的功能名称

# 4. 推送到远程仓库
git push origin master

# 5. 删除本地功能分支（可选）
git branch -d feature/你的功能名称

# 6. 删除远程功能分支（可选）
git push origin --delete feature/你的功能名称
```

**注意**：使用 `--no-ff` 参数可以保留分支的提交历史，使项目历史更清晰。

### 合并后自动部署

一旦 PR 合并到 master 分支，GitHub Actions 会自动：

1. 检出最新的代码
2. 安装依赖
3. 构建网站
4. 部署到 GitHub Pages

你可以在 "Actions" 标签下查看部署进度。部署完成后，更改会在几分钟内反映到 https://zhangwei8387.github.io 上。

## 常见问题

### Q: PR 检查失败怎么办？

A: 点击失败的检查，查看详细的错误日志。常见问题包括：
- 依赖安装失败：可能是 `package.json` 或 `package-lock.json` 有问题
- 构建失败：检查 Hexo 配置文件和内容文件是否有语法错误
- 修复问题后，推送新的提交到你的分支，检查会自动重新运行

### Q: 如何更新我的 PR？

A: 只需在你的功能分支上提交新的更改并推送：

```bash
git add .
git commit -m "修复问题"
git push origin feature/你的功能名称
```

PR 会自动更新，检查也会重新运行。

### Q: master 分支有新的提交，如何同步到我的 PR？

A: 在你的功能分支上合并或变基 master：

```bash
# 方式一：合并（推荐）
git checkout feature/你的功能名称
git fetch upstream
git merge upstream/master
git push origin feature/你的功能名称

# 方式二：变基（会重写历史）
git checkout feature/你的功能名称
git fetch upstream
git rebase upstream/master
git push origin feature/你的功能名称 --force-with-lease
```

### Q: 我没有合并权限怎么办？

A: 如果你是外部贡献者，创建 PR 后需要等待项目维护者审查和合并。你只需要：
1. 创建 PR
2. 确保所有检查通过
3. 等待维护者反馈
4. 根据反馈修改（如果需要）

### Q: 合并冲突怎么解决？

A: 如果 PR 显示有合并冲突：

1. 在本地同步 master 分支：
   ```bash
   git fetch upstream
   git checkout master
   git merge upstream/master
   ```

2. 切换到你的功能分支并合并 master：
   ```bash
   git checkout feature/你的功能名称
   git merge master
   ```

3. 解决冲突：
   - Git 会标记冲突的文件
   - 手动编辑这些文件，解决冲突
   - 使用 `git add` 标记为已解决
   - 提交合并：`git commit`

4. 推送到你的分支：
   ```bash
   git push origin feature/你的功能名称
   ```

## 联系方式

如果你有任何问题或建议，欢迎：
- 创建 Issue 讨论
- 在 PR 中提出问题
- 通过博客评论联系

感谢你的贡献！🎉
