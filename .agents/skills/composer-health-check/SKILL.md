---
name: composer-health-check
description: 检查当前仓库 PHP/Composer 项目的依赖安全公告、版本维护状态、自动加载和代码质量。当用户要求 Composer 健康检查、依赖审计或 PHP 项目质量审查时使用；普通功能开发无需启动全面审计。
license: MIT (see LICENSE)
---

# Composer 项目健康检查

输出有证据、按实际影响排序的健康报告；仅在用户要求修复时实施相应修改。

## 仓库与运行环境

- 本技能位于仓库根目录 `.agents/skills/composer-health-check/`，操作范围限于该仓库。
- 先读取适用的 `AGENTS.md` 和 Git 状态，保留已有改动。用户指定的项目路径优先；否则当前仓库默认检查 `webman/composer.json` 及同目录的 `composer.lock`。
- 若目录结构改变，用 `rg --files -g composer.json -g composer.lock -g '!**/vendor/**'` 定位项目；多个项目只在范围无法判断时询问。
- 若仓库根目录存在 `.codegraph/`，理解或定位源码前先使用 `codegraph_explore` 或 `codegraph explore`；不存在则跳过，不创建索引。
- 确认 PHP、Composer 的实际版本与运行位置。Windows 访问 WSL 项目时，优先使用该项目已有的 WSL PHP/Composer 环境，明确工作目录，不混用两端的 `vendor/`。
- 读取 PHP 与扩展约束、项目类型、`autoload`、`autoload-dev`、脚本和现有检查工具。`type: project` 视为应用；无 `type` 时结合用途判断，不一律当成库。

## 诊断

1. 在 Composer 项目目录执行适用的只读检查。以下为存在锁文件时的例子；统一禁用插件及项目脚本，避免诊断触发额外操作：

   ```text
   composer --no-interaction --no-ansi --no-plugins --no-scripts validate --no-check-publish
   composer --no-interaction --no-ansi --no-plugins --no-scripts audit --locked --format=json
   composer --no-interaction --no-ansi --no-plugins --no-scripts outdated --locked --direct --format=json
   ```

   对可发布的库保留发布元数据检查，省略 `--no-check-publish`。完整扫描可增加一次不带 `--direct` 的过期检查。
2. 没有锁文件时，只在已有安装依赖可用的情况下去掉 `--locked` 检查，并注明版本来源；否则报告无法确定实际依赖版本。不要为了诊断生成锁文件、安装依赖或安装检查工具。
3. 检查退出码、标准错误和 JSON 正文。安全公告或废弃包可能使审计返回非零；网络失败、工具缺失不能解释成“未发现风险”。旧版 Composer 不支持某选项时，查对应 `help` 后调整，或说明跳过原因。
4. 按 [依赖检查说明](references/dependency-checks.md) 核实公告、废弃包、版本约束和锁文件状态。只有用户提出 PHP 升级目标时，才针对该版本执行 `composer prohibits php TARGET_VERSION`，不要猜测“最新稳定版”。
5. 从实际业务源码抽查代表性文件，必要时用 `rg` 扩大搜索；按 [代码质量说明](references/code-quality-checks.md) 检查兼容性、自动加载、错误处理、类型契约和明显的资源问题。排除 `vendor/`、生成代码及无关的 skillbox 示例；不要把 PSR-4 的空前缀机械扩展为整仓通读。
6. 只运行项目已有、与问题相关且不会修改文件的检查。报告已检查的目录、抽样范围和遗漏，不把抽样审查说成全量验证。

快速模式执行适用的验证、安全公告、直接依赖检查，抽查 3–5 个业务文件，再给出简短报告。

## 报告与修复

- 列出项目路径、PHP 约束与运行版本、版本数据来自锁文件还是安装目录、检查范围和执行限制。
- 每项发现包含优先级、包及版本或文件行号、证据、实际影响、建议；相同问题合并，列出代表位置和可信的数量。
- 按公告提供的严重度与实际业务影响排序。缺少 `strict_types`、可选类型声明或新语法通常属于建议；过期不等于存在漏洞，间接依赖也不能天然降级。
- 没有发现问题时说“在本次已完成检查的范围内未发现问题”。检查失败或被跳过时，不宣称依赖安全或已是最新。
- 用户只要求检查时，完成报告并给出可选修复项。已经要求“检查并修复”时，在既有授权范围内继续，不重复要求批准；升级目标或破坏性改动尚不明确时才澄清。
- 依赖修改使用有针对性的 Composer 命令，审查 `composer.json` 与锁文件 diff，保留项目需要的安装脚本流程。不要手工伪造锁文件或默认执行全量更新；库的兼容范围与应用的锁定策略分别判断。
- 修复后运行相关验证。涉及依赖公告的修复重新执行审计，准确报告仍存在的公告及验证限制。

改编自 [skillbox/composer-health-check](https://github.com/zxc7563598/skillbox/tree/main/skills/composer-health-check)，保留原作者 [MIT 许可](LICENSE)。
