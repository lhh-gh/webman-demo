---
name: git-commit
description: 根据当前仓库的实际 Git 改动生成 Conventional Commits 提交信息，英文 type 与中文描述；用户明确要求提交时完成所选改动的提交。用于提交代码、生成提交信息或整理提交；仅生成信息时不执行提交。
license: MIT (see LICENSE)
---

# Git Commit

理解改动目的，生成准确的中文提交信息，并尊重用户指定的提交范围。

## 检查状态与范围

1. 本技能作用于包含 `.agents/skills/git-commit/` 的当前仓库。读取适用的 `AGENTS.md`，确认仓库根目录和分支。Git 根目录与 Composer 子项目目录可能不同。
2. 获取以下只读信息；必要时补充受影响文件的上下文：

   ```text
   git status --short --untracked-files=normal
   git diff --no-ext-diff --no-color
   git diff --cached --no-ext-diff --no-color
   git diff --cached --name-status
   ```

   未跟踪文件不在 diff 中；用 `git ls-files --others --exclude-standard` 列出候选，再读取属于本次任务的文件。较大 diff 先看统计和分组，不以截断输出代替审查。
3. 用户明确指定的文件、已批准的清单或当前任务范围优先。只要求提交暂存区时保留其原样。存在暂存与未暂存混合改动时，不能默认全部提交；范围无法从会话判断才询问。
4. 提交会包含整个暂存区。准备提交前确认其中没有范围外改动；若存在，不擅自取消暂存，先明确处理方式。同一文件既有暂存又有未暂存内容时，不直接 `git add` 覆盖用户挑选的 hunks；只有用户明确要求提交该文件全部内容时才整文件暂存。
5. 没有可提交内容时直接说明。合并冲突未解决时先报告；不要绕开问题生成一个看似成功的提交。

## 编写信息

格式与类型见 [Conventional Commits 参考](references/conventional_commits.md)：

```text
type(scope): 中文主题

需要时用中文解释改动目的、行为变化和影响。
```

- type 使用小写英文；scope 可选，优先沿用仓库约定。主题简短、动宾结构、无句末句号，通常控制在 50 个字符内；不要机械按字节长度判断中文。
- 根据实际目的区分 `feat`、`fix`、`refactor`、`perf`、`docs`、`style`、`test`、`build`、`ci`、`chore`。不要只从新增或删除行数猜测意图。
- 简单改动可只有主题；正文解释为什么和重要影响，不重复 diff。未经执行的检查不能写成已通过，未知的动机不能编造。
- 有证据的破坏性变更使用 `!` 或 `BREAKING CHANGE:`。不捏造 issue 关联、审核记录或署名；不默认添加 AI 署名。用户或仓库明确要求的合法 trailers、签名约定优先。
- 独立主题可以建议拆分，但不擅自扩大提交数量或范围。

## 展示与提交

1. 提交前用普通文本展示完整提交信息和带状态的文件清单，让用户能够审阅。
2. 用户只要求“生成提交信息”时，到此完成。用户已明确要求提交或批准了范围时，展示后按授权继续，无需重复确认。仅当提交意图、范围或必须遵循的仓库要求不明确时，提出具体澄清。
3. 需要暂存时使用 `git add -- PATH...`，路径明确且正确引用；不要默认 `git add -A`。重新核对暂存 diff 与文件清单，再执行相关已有检查。
4. 将展示的原文写入 UTF-8 临时文件，使用 `git commit --file MESSAGE_FILE`，完成后清理自己的临时文件。不要将多行正文拼进 shell 字符串，尤其在 PowerShell 中避免 `$()` 和反引号被解释。
5. 提交失败时展示真实错误并保留工作。检查或 hook 失败时针对原因处理；若 hook 修改了文件，重新审查且不要自动扩大暂存范围。不要默认使用 `--no-verify`、`--amend`、更改 Git 身份、关闭签名或推送。
6. 成功后读取 `git log -1 --format=fuller` 和 `git status --short`，报告提交 hash、分支、主题及剩余改动。准确区分已运行、未运行和失败的检查。

改编自 [skillbox/git-commit](https://github.com/zxc7563598/skillbox/tree/main/skills/git-commit)，保留原作者 [MIT 许可](LICENSE)。
