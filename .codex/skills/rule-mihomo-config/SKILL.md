---
name: rule-mihomo-config
description: 安全维护本仓库的 Mihomo/Clash Meta 配置。用于编辑、审查、解释、验证或提交 config/mihomo 下的文件，尤其是 config/mihomo/me/my-configdns.yaml、proxy-groups、DNS、TUN、rule-providers、节点筛选、故障转移、自动测速、订阅 provider 等相关改动。
---

# Rule Mihomo Config

## 工作流程

1. 修改前先确认上下文：
   - 优先读取 `config/mihomo/me/my-configdns.yaml`。
   - 需要参考模板时，再对比 `config/mihomo/*.yaml`。
   - 用 `rg` 搜索策略组名、规则、YAML anchor、provider 引用。

2. 保持改动范围小：
   - 用户没有明确要求时，优先只改 `config/mihomo/me/my-configdns.yaml`。
   - 不要随意重命名中文策略组名，因为 rules、filter、用户习惯都依赖这些名字。
   - 不要为了“看起来完整”新增大量 rule-providers 或策略组；本项目目标是精简好用。

3. 修改后验证：
   - 运行 `git diff --check -- config/mihomo/me/my-configdns.yaml`。
   - 如果本机存在 `mihomo`、`clash` 或 `clash-meta`，再运行可用的配置检查命令。
   - 如果没有内核命令或 YAML 解析器，要明确说明只做了静态检查。
   - 检查每个 `RULE-SET` 都有同名 `rule-provider`，并检查不同 provider 是否误用了相同 URL；重复 URL 常见于复制粘贴错误。
   - 新增远端规则时读取文件开头确认实际格式，不要根据 `.txt`、`.yaml` 等扩展名推断 `format`。
   - 删除策略组后，用 `rg -n "组名" config/mihomo/me/my-configdns.yaml` 检查是否还有残留引用。

4. 用户要求提交时：
   - 先运行 `git status --short`，确认只提交用户想要的文件。
   - 本环境可能遇到 `fatal: detected dubious ownership`。不要写用户全局 Git 配置，使用临时 config：

```powershell
$cfg = Join-Path $env:TEMP 'codex-rule-gitconfig'
$env:GIT_CONFIG_GLOBAL = $cfg
git config --global --add safe.directory 'F:/workspace/workspace02/rule'
git status --short
```

   - `.git` 在沙箱里可能只读，`git add` 和 `git commit` 可能需要提权。
   - 如果 Git 仓库自动发现失败，可显式指定仓库：

```powershell
git --git-dir='.git' --work-tree='.' status --short
git --git-dir='.git' --work-tree='.' diff --check -- config/mihomo/me/my-configdns.yaml
```

   - 如果 Git 缺少作者身份，优先复用最近一次提交作者，并只对本次 commit 临时指定：

```powershell
git log -1 --format='%an <%ae>'
git -c user.name='小章鱼鸭' -c user.email='47490129+zykzhangyukang@users.noreply.github.com' commit -m '...'
```

## 持续维护约定

- 后续在这个仓库里遇到新的 Mihomo 配置知识、踩坑、验证方法、Git 环境问题，应该同步补进这个 skill。
- 只记录对以后有复用价值的内容，不记录一次性闲聊或无关细节。
- 新增内容要短、具体、可操作；优先写“以后应该怎么做”和“为什么容易踩坑”。
- 如果本 skill 和当前实际配置冲突，以实际文件为准，并更新本 skill。

## 当前配置约定

- 主配置文件是 `config/mihomo/me/my-configdns.yaml`。
- 静态直连代理名是 `直连`，不要随意改名。
- 订阅 provider 当前使用：
  - `type: http`
  - `interval: 86400`
  - 健康检查 `https://www.gstatic.com/generate_204`
  - `proxy: 直连`
- 订阅 URL 里有 token。公开展示、迁移到文档、复制到示例前要提醒用户。
- 全局默认配置偏安静和实用：
  - `log-level: warning` 只输出警告和错误，不记录大量普通连接细节。
  - `ipv6: false`。
  - `unified-delay: true`。
  - `tcp-concurrent: true`。
- TUN 当前把 `auto-route`、`auto-redirect`、`auto-detect-interface` 设为 `false`，适合 Nikki 或前端接管路由的场景。裸 Mihomo 内核运行时，这几项通常需要改成 `true`。
- DNS 当前使用 fake-ip 和 `respect-rules: true`，不要随便改；只有在排查 DNS 泄露或兼容性问题时再动。

## 策略组模式

推荐保持日常策略列表精简：

```yaml
[自动选择, 故障转移, 狮城节点, 日本节点, 香港节点, 美国节点, 全部节点, 直连]
```

用 YAML anchor 复用重复列表：

```yaml
proxies: &common-proxies [...]
proxies: *common-proxies
```

注意：
- anchor 必须先定义，再引用。
- `*common-proxies` 和 `*node-exclude` 是 YAML 别名，不是正则表达式。
- 默认不要重新添加 `其他节点`。`全部节点` 已经能作为所有节点的兜底入口。

## 节点筛选

地区策略组保持简单：

- `香港节点`：匹配香港关键词。
- `日本节点`：匹配日本关键词。
- `狮城节点`：匹配新加坡关键词。
- `美国节点`：匹配美国关键词。
- `全部节点`：使用 `include-all: true`。

共享的 `node-exclude` 用来从地区组里排除状态节点和特殊节点：

```regex
(?i)(剩余|流量|官网|套餐|到期|过期|Expire|Traffic|Reset|Game|游戏|家宽|倍率|[0-9]+(\.[0-9]+)?x|[0-9]+(\.[0-9]+)?倍)
```

含义：
- `(?i)` 表示大小写不敏感。
- 排除剩余流量、套餐到期、官网、重置时间等伪节点。
- 默认排除游戏节点和家宽节点。
- 默认排除倍率节点，例如 `0.5x`、`2x`、`1倍`、`3倍`，避免自动分组误用特殊计费节点。

如果用户想把高质量专线或低倍率节点放回普通分组，只放宽倍率相关部分，不要整段删除 `node-exclude`。

## 自动选择和故障转移

- `自动选择` 使用 `type: url-test`，按延迟选择节点。当前低噪声设置是 `tolerance: 50` 和 `interval: 600`。
- `故障转移` 使用 `type: fallback`，按顺序使用第一个健康节点；当前节点不可用时切到下一个健康节点。
- `fallback` 配合 `include-all: true` 时，排序通常来自：
  1. `proxy-providers` 的顺序。
  2. 每个订阅内部的节点顺序。
  3. 再经过 `filter` 和 `exclude-filter` 过滤。
- 如果用户想让故障转移按地区优先，例如狮城、日、港、美，要显式写 `proxies`：

```yaml
proxies: [狮城节点, 日本节点, 香港节点, 美国节点, 全部节点]
```

只有用户明确要求固定地区优先级时，才改成这种写法。

## 规则和 Provider 清理

- 保持 rules 精简，优先使用现有组：`默认代理`、`ChatGPT`、`GitHub`、`YouTube`、`Google`、`TikTok`、`Telegram`、`NETFLIX`、`直连`、`漏网之鱼`。
- `behavior` 描述规则语义（`domain`、`ipcidr`、`classical`），`format` 描述文件编码（`yaml`、`text`、`mrs`），两者不要混淆。
- YAML merge key 提供默认值，同一映射中的显式字段会覆盖锚点字段。例如 `{ <<: *class, format: yaml, url: ... }` 会覆盖 `*class` 的 `format: text`。
- Loyalsoldier release 中扩展名为 `.txt` 的规则可能实际以 `payload:` 开头，属于 `format: yaml`；必须检查远端内容后配置。
- 当前 Mihomo 可省略 HTTP rule-provider 的 `path` 并自动管理缓存；只有兼容要求显式路径的旧 Clash 内核时才保留。
- 新增 provider 后必须在 `rules` 中添加 `RULE-SET` 引用，否则 provider 不参与分流。`ipcidr` 规则通常配合 `no-resolve`。
- 专用服务规则应放在通用代理规则前面，否则通用 provider 可能提前命中，使专用策略组失效。
- `applications` 已包含常见下载工具的 `PROCESS-NAME` 时，不要再保留同目标、同策略的重复进程规则。
- 删除某条规则后，如果对应 `rule-provider` 没有其他引用，也要一起删除。
- `MATCH,漏网之鱼` 必须放最后。
- private IP/domain 的直连规则应放在代理规则前面。

## 常见解释

- `log-level: warning` 是 Mihomo 内核运行日志级别，不是普通访问日志。它输出 warning 和 error，不输出大量连接过程。
- `fallback` 偏稳定和顺序容灾；`url-test` 偏低延迟选择。
- `allow-lan: true` 允许局域网设备连接这台机器的代理；不需要手机或其他设备访问时，可以建议改成 `false`。
