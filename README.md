# justincourse-skills

> **开发中，服务尚未上线。** 现在安装本仓库的插件或 skill 也连不上服务，这不是插件坏了；服务正式上线后会去掉这条标注。

JustinCourse 课程助手的公开仓库：让 AI 助手（Claude Code、Codex、Cursor、claude.ai 等）通过 MCP 读取你在 [justincourse.com](https://justincourse.com) 已有权限的课程内容。

*English summary: client config and a usage skill for the (not yet live) justincourse.com MCP server at `https://justincourse.com/api/mcp`. Read-only; signs in with your own justincourse.com account via OAuth.*

## 这是什么

- 一个 MCP 服务：`https://justincourse.com/api/mcp`（无状态 Streamable HTTP，用你自己的 justincourse.com 账号通过 OAuth 登录授权）。
- 一个使用流程 skill（`skills/justincourse-courses/SKILL.md`）：告诉模型怎么定位课时、翻页、引用来源、处理锁定内容。
- 一个 Claude Code 插件 + marketplace 清单，一条命令装好服务配置和 skill。

工具名称、参数与返回结构以服务端 `tools/list` 为准，本仓库不复述。

## 能做什么

- 列出你有权限的课程，查看课程目录；
- 读取课时的正文和字幕（较长时分页读取）；
- 按你的权限搜索课程内容；
- 围绕课程内容提问，回答会引用课时标题并附课时链接。

免费用户只能读到课程目录、免费预览课时和图文试读段；会员按自己有权限的课程读取全文与字幕。具体口径以 [justincourse.com/skills](https://justincourse.com/skills) 介绍页为准。

## 内测说明

目前是内测阶段，**不含在任何套餐里，也没有任何公开承诺**（包括功能范围、价格、上线时间）。内测期间介绍页 [justincourse.com/skills](https://justincourse.com/skills) 也还没有上线，上线前打不开属正常。

## 接入方法

核实日期 2026-11-08：下面各客户端的配置写法逐一对照了当时的官方文档（链接见每一节），并在本机用 `claude` 2.1.296、`codex` 0.166.0 的 `--help` 核对了命令行参数。客户端更新很快，如与你的版本不符，以官方文档为准。以下所有方式都需要服务上线后才能真正连上。

### Claude Code

**方式一：插件（含服务配置 + skill）**

```bash
claude plugin marketplace add justincourse/justincourse-skills
claude plugin install justincourse@justincourse-skills
```

在会话里也可以用 `/plugin marketplace add justincourse/justincourse-skills`，再 `/plugin install justincourse@justincourse-skills`。装好后在会话里运行 `/mcp`，选择 `plugin:justincourse:justincourse`，按浏览器提示登录授权。

**方式二：只添加 MCP 服务（不带 skill）**

```bash
claude mcp add --transport http justincourse https://justincourse.com/api/mcp
```

加 `--scope user` 可在你所有项目里可用。然后在会话里 `/mcp` 选择该服务完成登录；也可以在终端运行 `claude mcp login justincourse`。

文档：<https://code.claude.com/docs/en/mcp>、<https://code.claude.com/docs/en/plugins/install.md>、<https://code.claude.com/docs/en/plugin-marketplaces>

### Codex

在 `~/.codex/config.toml`（信任的项目里也可用 `.codex/config.toml`）添加：

```toml
[mcp_servers.justincourse]
url = "https://justincourse.com/api/mcp"
```

或用命令添加：

```bash
codex mcp add justincourse --url https://justincourse.com/api/mcp
```

然后登录授权（无本地浏览器的终端加 `--no-browser`）：

```bash
codex mcp login justincourse
```

文档：<https://developers.openai.com/codex/mcp>（会跳转到 OpenAI 的新文档地址）。

未能核实：Codex 是否需要额外的实验开关才能启用 OAuth 的 HTTP 服务（官方文档没有提到需要）；`codex mcp add` 的 OAuth 动态注册行为，官方示例只展示了带 `--oauth-client-id` 的写法，本服务支持动态客户端注册，应当不需要；以及 Codex 的插件 / skill 安装入口是否能直接使用本仓库，本仓库未针对 Codex 插件格式做适配。skill 可以用下面的通用方式安装。

### Cursor

在 `~/.cursor/mcp.json`（全局）或项目内 `.cursor/mcp.json` 添加：

```json
{
  "mcpServers": {
    "justincourse": {
      "url": "https://justincourse.com/api/mcp"
    }
  }
}
```

Cursor 对需要登录的远程服务使用授权码 + PKCE 的 OAuth 流程，默认动态注册客户端；首次使用时按提示在浏览器里登录授权。

文档：<https://cursor.com/docs/context/mcp>。未能核实：本机没有安装 Cursor，没有实际连接测试；文档未给出可用的「一键添加」链接格式，所以这里不提供。

### claude.ai（自定义连接器）

在 claude.ai 的 **Customize > Connectors** 里点 **+ Add**，选 **Add custom connector**，名称随意，服务地址填 `https://justincourse.com/api/mcp`，按提示继续；鉴权选择需要登录的方式，OAuth 客户端选官方推荐项或自动注册均可，之后在浏览器里登录 justincourse.com 授权。在对话里点输入框左下角的 **+**，选 **Connectors**，打开 justincourse 的开关即可。

官方文档说明：自定义连接器在 Free、Pro、Max、Ultra、Team、Enterprise 套餐上均可添加（Free 限一个）；Team 只有 Owner 能在组织设置里添加，Enterprise 取决于角色；连接请求从 Anthropic 云端发出，所以服务必须能被公网访问。

文档：<https://support.claude.com/en/articles/11175166-get-started-with-custom-connectors-using-remote-mcp>。未能核实：界面菜单名称随版本调整较多，以你当前看到的为准；本服务的 OAuth 在 claude.ai 上的实际表现要等服务上线后才能验证。

### 只安装 skill（通用方式）

支持 [`skills`](https://github.com/vercel-labs/skills) 命令行工具的客户端，可以只装使用流程 skill（不含服务配置，服务仍需按上面的方式单独接入）：

```bash
npx skills add justincourse/justincourse-skills
```

该工具会在仓库的 `skills/` 目录下查找 `SKILL.md`（对照其文档 <https://github.com/vercel-labs/skills> 核实；未实际执行安装）。

## 版本对应

| 本仓库版本 | MCP 服务端 `serverInfo.version` |
| :--- | :--- |
| 0.1.0 | 0.1.0 |

服务端 `serverInfo` 为 `{ name: "justincourse", version: "0.1.0" }`。变更记录见 [CHANGELOG.md](./CHANGELOG.md)。

## 隐私与权限

- **只读**：所有工具都只读取内容，不会修改你的账号、订单或学习记录。
- **只读你已有权限的内容**：与你登录网站时能看到的一致；没有权限的课时只会返回锁定信息，不会返回正文。
- **你来控制授权**：授权范围为 `courses:read offline_access`。随时可以在 justincourse.com 账户页的「已授权应用」（`/authorized-apps`）撤销，撤销后该客户端不能再访问。
- 服务端会为安全与限流记录调用日志，具体说明以介绍页为准。
- 本仓库不包含任何付费内容、测试账号、密码或密钥。

## 相关链接

- 站内介绍页：<https://justincourse.com/skills>（**尚未上线**，上线前打不开属正常）
- 网站：<https://justincourse.com>

## 许可证

[MIT](./LICENSE)
