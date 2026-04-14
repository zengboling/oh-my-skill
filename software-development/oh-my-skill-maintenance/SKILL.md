---
name: oh-my-skill-maintenance
description: 维护 oh-my-skill 仓库的 Skill - 用于将本地 Hermes Skills 同步至 GitHub 备份仓库
trigger: 当用户要求将某个 Skill 添加到 oh-my-skill 仓库，或更新仓库中的 Skill 时
---

# oh-my-skill 仓库维护 Skill

本 Skill 定义了将本地 `.hermes/skills` 目录下的 AI Skill 同步到 `oh-my-skill` Git 仓库的标准流程，确保备份的一致性与结构完整。

## 核心路径定义
- **本地源路径**: `~/.hermes/skills/`
- **仓库目标路径**: `~/code/rewbl/oh-my-skill/`

## 操作指令

### 1. 添加/同步单个 Skill
当用户指定某个 Skill 需要同步时，执行以下步骤：

1. **定位源文件**: 
   - 在 `~/.hermes/skills/` 中搜索该 Skill 的目录。
   - 确认其所属的分类目录（Category）。
2. **镜像目录结构**:
   - 在 `~/code/rewbl/oh-my-skill/` 下创建相同的分类目录结构。
   - 例如：如果 Skill 位于 `~/.hermes/skills/devops/opensource-deploy/`，则目标路径应为 `~/code/rewbl/oh-my-skill/devops/opensource-deploy/`。
3. **完整拷贝**:
   - 将该 Skill 目录下的所有文件（包括 `SKILL.md` 及 `references/`, `templates/`, `scripts/` 等子目录）全部拷贝到目标路径。
4. **提交变更**:
   - 执行 `git add .`
   - 提交信息格式: `feat: add/update [skill-name] skill`
   - 执行 `git push`

### 2. 批量同步/审计
当用户要求同步所有 Skill 或检查差异时：

1. **目录对比**: 
   - 对比 `~/.hermes/skills/` 与 `~/code/rewbl/oh-my-skill/` 的目录树。
2. **补全缺失**: 
   - 将本地存在但仓库缺失的 Skill 按照上述“单个同步”流程全部添加。
3. **更新过时**: 
   - 将本地更新日期更晚的 `SKILL.md` 覆盖到仓库中。
4. **提交变更**:
   - 提交信息格式: `chore: sync all skills from local store`

## 注意事项
- **排除项**: 不要同步 `.hub/` 目录或任何临时缓存文件。
- **权限验证**: 确保使用 `gh auth setup-git` 配置 Git 凭据，避免 push 时出现交互式密码提示。
- **验证**: 同步完成后，可以通过 `ls -R` 检查目标路径，确保 `SKILL.md` 文件已正确到位。
