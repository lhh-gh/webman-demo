# 仓库协作说明

本文件适用于整个 Git 仓库；进入子目录工作时，同时遵循该路径下更具体的 `AGENTS.md` 或 `AGENTS.override.md`。

## 项目与目录

这是基于 Webman 2.x / Workerman 的 PHP 应用。根目录的 `composer.json` 声明 PHP `>=8.1`，使用 Composer 管理依赖和自动加载。

**Git 根目录就是应用目录。** Git、PHP、Composer 和应用运行命令默认在当前仓库根目录执行。

| 路径（相对仓库根目录） | 用途 |
| --- | --- |
| `app/controller/` | HTTP 控制器，使用 `support\Request` 接收请求 |
| `app/middleware/` | 中间件 |
| `app/model/` | 数据模型 |
| `app/process/` | HTTP 进程与文件、内存监控进程 |
| `app/view/` | 页面模板 |
| `config/` | 路由、进程、应用、日志和其他配置 |
| `support/` | 应用支持代码与启动逻辑 |
| `public/` | 公共静态资源 |
| `start.php` | Linux / WSL 启动入口 |
| `windows.php`、`windows.bat` | 原生 Windows 启动入口 |
| `composer.json`、`composer.lock` | 依赖声明与锁定版本 |
| `.agents/skills/` | 本仓库的 Codex 技能及其资源 |

`vendor/` 是安装依赖，`runtime/` 是日志等运行产物；它们不属于日常修改的源码范围。

## 开发与运行

当前工作区通过 Windows 访问 WSL 文件系统。优先使用项目所在 WSL 环境的 PHP 和 Composer；保持命令与工作目录在同一环境中。不要将 PowerShell 路径直接传给 Linux 命令，或把 Bash 语法直接交给 PowerShell。

以下命令均从仓库根目录执行，按任务需要选用：

```sh
# 初次配置开发环境：按锁文件安装依赖
composer install

# 前台开发运行，默认监听 http://0.0.0.0:8787
php start.php start

# 查询、重载或停止已有服务
php start.php status
php start.php reload
php start.php stop
```

- 原生 Windows 使用 `php windows.php` 或现有 `windows.bat`，不要套用 Linux 的进程管理方式。
- HTTP 监听地址与进程数配置在 `config/process.php`。Linux / WSL 的前台模式默认监控 PHP、HTML 和 env 文件；守护模式 `start -d` 不启用该文件监控。
- 服务进程常驻内存，避免将请求数据保存在跨请求共享的静态属性或全局变量中。涉及入口、依赖或进程配置的改动，按实际重载范围判断是否需要重启。
- 存在 `Dockerfile` 和 `docker-compose.yml`；只有任务需要容器运行时才使用并验证它们，不将模板存在等同于构建成功。

## 修改约定

- 沿用相邻文件的 PHP 格式、命名空间和目录组织。现有业务命名空间主要为 `app\`，基础支持类为 `support\`；注意 WSL/Linux 路径大小写。
- 新代码兼容声明的最低 PHP 8.1 版本。不要仅因本机版本较新就采用 PHP 8.2+ 语法，除非任务明确包含提升兼容基线。
- 遵循既有控制器、请求、响应和配置约定；类型声明或 `strict_types` 的调整须考虑框架接口和调用兼容性，不为风格统一批量改动无关文件。
- 依赖变更使用 Composer，并审查 `composer.json` 与锁文件差异；应用项目保留锁文件，不手工修改其解析结果。常规安装使用 `install`，不要顺带全量 `update`。
- 保持已有换行符与可执行位；避免整文件换行转换或无意改变 `start.php` 的文件模式。
- `.idea/` 已由根目录忽略规则排除。遵循应用的忽略规则，不提交 `vendor/`、`runtime/`、`.env` 等本地产物。

## 验证

- 修改 PHP 后，对实际修改的文件运行 `php -l`；例如在仓库根目录运行 `php -l app/controller/IndexController.php`。
- 当前未发现项目级 PHPUnit、PHPStan、Psalm 或代码风格检查配置，`composer.json` 也未定义测试脚本。不要报告不存在的测试命令已通过；新增相关配置后更新本说明。
- 修改依赖声明时，在仓库根目录执行 `composer --no-interaction --no-ansi --no-plugins --no-scripts validate --no-check-publish`；涉及依赖安全或维护状态时使用下方健康检查技能。
- 行为改动按范围验证路由、响应、配置或进程逻辑。需要运行服务时，先确认现有进程与端口，保留用户启动的服务，并清理自己启动的临时进程。
- 纯文档或技能调整检查文件引用与适用格式即可；最终说明实际完成的验证，以及无法运行的检查和原因。

## 协作与本地技能

- 默认用中文沟通。开始修改前查看 Git 状态，保留用户已有改动；已明确授权的工作继续完成，仅在信息缺口影响结果时澄清。
- 只有用户明确要求时才提交或推送。提交范围以用户指定和当前任务为准，避免夹带其他暂存或未暂存改动。
- 以下技能位于本仓库 `.agents/skills/`；匹配任务时先读取对应说明，具体流程不在本文件重复维护：
  - [composer-health-check](.agents/skills/composer-health-check/SKILL.md)：Composer 依赖与 PHP 项目健康检查，默认检查仓库根目录。
  - [git-commit](.agents/skills/git-commit/SKILL.md)：按实际改动生成英文 type、中文描述的 Conventional Commits 信息，并在授权时提交。
  - [think-council](.agents/skills/think-council/SKILL.md)：复杂方案的多视角评估，可完整输出或按用户要求逐轮讨论。
  - [webman-development](.agents/skills/webman-development/SKILL.md)：Webman 应用开发、调试与验证，按任务加载对应参考文档。

<!-- CODEGRAPH_START -->
## CodeGraph

In repositories indexed by CodeGraph (a `.codegraph/` directory exists at the repo root), reach for it BEFORE grep/find or reading files when you need to understand or locate code:

- **MCP tool** (when available): `codegraph_explore` answers most code questions in one call — the relevant symbols' verbatim source plus the call paths between them, including dynamic-dispatch hops grep can't follow. Name a file or symbol in the query to read its current line-numbered source. If it's listed but deferred, load it by name via tool search.
- **Shell** (always works): `codegraph explore "<symbol names or question>"` prints the same output.

If there is no `.codegraph/` directory, skip CodeGraph entirely — indexing is the user's decision.
<!-- CODEGRAPH_END -->
