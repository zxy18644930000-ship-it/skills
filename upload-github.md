将当前目录的代码自动上传到 GitHub。

请按以下步骤执行，全程自动化，不要询问确认：

## 步骤

1. **检查当前目录状态**
   - 运行 `git status` 检查是否已经是 git 仓库
   - 如果不是 git 仓库，执行 `git init`

2. **检查 .gitignore**
   - 如果没有 .gitignore 文件，根据项目语言/框架自动创建一个合理的 .gitignore（参考 GitHub 官方模板）
   - 确保不会提交敏感文件（.env、credentials、node_modules 等）

3. **提交所有代码**
   - `git add .`（排除 .gitignore 中的内容）
   - 分析代码内容，生成一个有意义的 commit message
   - 执行 `git commit`

4. **创建 GitHub 远程仓库**
   - 用 `gh repo list` 检查是否已有同名远程仓库
   - 如果没有远程仓库，用 `gh repo create` 创建一个（默认 public，仓库名用当前目录名）
   - 如果已有远程 origin，跳过创建

5. **推送代码**
   - 设置远程 origin（如果还没设置）
   - `git push -u origin main`（或当前分支）

6. **输出结果**
   - 显示 GitHub 仓库 URL
   - 显示推送状态

## 注意
- 如果 `gh` CLI 未安装或未登录，提醒用户先执行 `gh auth login`
- 遇到错误时尝试自动修复，实在无法解决再告知用户
