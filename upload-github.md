将当前目录的代码自动上传到 GitHub，同时同步技能和知识库到 skills 仓库。

请按以下步骤执行，全程自动化，不要询问确认：

## 第一部分：推送当前项目代码

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

## 第二部分：同步技能和知识库到 skills 仓库

每次推送完当前项目代码后，自动执行以下操作：

1. **复制最新文件到 skills 目录**
   ```
   # 技能文件
   cp ~/.claude/commands/*.md ~/skills/
   # 知识库
   cp ~/Scripts/price_sum_knowledge.json ~/skills/
   cp ~/Scripts/price_sum_pairs.json ~/skills/
   # Claude 配置和记忆（换电脑恢复用）
   mkdir -p ~/skills/claude-config/memory
   cp ~/.claude/CLAUDE.md ~/skills/claude-config/
   cp ~/.claude/settings.json ~/skills/claude-config/
   cp ~/.claude/projects/-Users-zhangxiaoyu/memory/*.md ~/skills/claude-config/memory/
   ```

2. **检查是否有变更**
   - `cd ~/skills && git status`
   - 如果没有变更，跳过后续步骤，输出"技能库无变更"

3. **提交并推送**
   - `git add .`
   - 生成 commit message，简要说明哪些文件有更新
   - `git commit && git push`

4. **输出结果**
   - 显示 skills 仓库 URL: https://github.com/zxy18644930000-ship-it/skills
   - 显示同步了哪些文件

## 输出汇总

最后输出两个仓库的推送结果：
- 项目仓库: URL + 状态
- 技能仓库: URL + 状态

## 注意
- 如果 `gh` CLI 未安装或未登录，提醒用户先执行 `gh auth login`
- 遇到错误时尝试自动修复，实在无法解决再告知用户
- skills 仓库已存在于 `~/skills/`，remote 已配置好，直接 push 即可
