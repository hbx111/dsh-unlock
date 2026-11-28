# dsh-unlock

把系统提示词里的严格内容政策换成宽松政策，并在末尾挂一段兜底段。它只改提示词文本：
不动宿主文件、不写会话记录、不改模型设置，关掉后下一次装配就恢复原样。

## 它做两件事

1. `systemPrompt.section()` 挂两段提示词：
   - `unlock:content-policy`（order 10，人设前缀之后）：宽松内容政策；
   - `unlock:priority-overlay`（order 10250，最末）：`<用户至上总纲>` 兜底段。
2. `system-prompt/assemble` 瀑布：把别处带来的严格 `<content_policy>…</content_policy>`
   就地换成同一份政策；按内容判重，保证政策与兜底各只有一份。

## 安装

组合包（bundle），要求 DSH `>=0.2.1-alpha.5 <0.3.0-0`（`engines.dsh`，版本不符时安装会被拒）。

这个区间覆盖 0.2.1-alpha.5 起、0.3.0 之前的所有版本：0.2.1 的 alpha/rc 线、0.2.2 的 alpha/rc 线与正式版、0.2.3 的 alpha 与正式版。0.2.0 及更早、0.2.1-alpha.4 及更早、0.3.x（含 0.3.0 的预发布版，这就是 `<0.3.0-0` 里 `-0` 的作用）会被拒。

下界取在 0.2.1-alpha.5 是因为 DSH 从这一版起才把 `engines.dsh` 纳入校验（更早的版本只校验 `peerDependencies`，声明了也不生效）。插件本身只用到 `systemPrompt.section()` 与 `system-prompt/assemble` 瀑布这两个稳定接口，实测从 0.1.7-rc.2 到 0.2.3-alpha.2 的每一版都挂得上、装配结果一致。

```sh
# 方式一：让 pnpm 直接从 GitHub 取（github: 规格由 pnpm 支持，本项目未实测这条路径）
dsh plugin --profile <profile> add github:hbx111/dsh-unlock

# 方式二：clone 下来按本地目录装（装的是 link，改完重载即可，不必重装）
git clone https://github.com/hbx111/dsh-unlock
dsh plugin --profile <profile> add ./dsh-unlock
```

`dsh --profile <profile> --dump-config` 里出现 `# == dsh-unlock` 层即为生效；
Web GUI 里对应「插件」页的组合包行，开关改在 profile 补丁层的同名 id 覆盖里。

## 文本来源（每次装配按 mtime 重读，改完下一次请求即生效）

```
~/.dsh/unlock/policy.txt   存在时整份替换内置政策（不给则用内置那份原文）
~/.dsh/unlock/prompt.txt   前置到政策开头，跳过空行与 # 注释行
~/.dsh/unlock/unlock.log   加载与异常记录，超过 256 KB 自动转 .1
```

## 开关（本组合包 cordis.patch.yml 的那一行 config）

| 字段 | 缺省 | 含义 |
|---|---|---|
| `enabled` | `true` | `false` = 什么都不注册 |
| `overlay` | `true` | `false` = 只换政策块，不追加末尾兜底 |
| `replaceStrictBlock` | `true` | `false` = 只挂自己的政策段，不改别处的政策块 |
| `log` | `true` | `false` = 不写 unlock.log |

profile 补丁层写同名 id 覆盖会整段替换 config，四个字段要写全。

## 边界

- 只动系统提示词：不碰宿主文件、会话记录、模型设置；关掉后下一次装配恢复原样。
- 压不过整段替换 system 的网关（自带整份提示词、以 `prompt.mode=custom` 整体覆盖的那种）。
- 不往用户消息里追加内容；需要那一层时用 `dsh-policy` 的 `mode=user`。
- 新会话立即带上；已在跑的会话在下次装配时收到更新通知，旧消息位置不动。

## 卸载

在插件页关掉或移除此组合包即可，`~/.dsh/unlock/` 里的文本与日志可留可删。

## 仓库结构

```
index.js            插件入口：两段 section + assemble 瀑布
cordis.patch.yml    组合包 patch，一行 insert，四个开关在 config 里
locale/zh.json      插件页标题与说明
icon.svg            插件图标
docs/               详情页配图（DETAIL.md / DETAIL.zh.md / preview.svg）
```

## 许可

MIT，见 [LICENSE](./LICENSE)。内置政策与兜底段原文随仓库公开，可自行改写
（`~/.dsh/unlock/policy.txt` 整份替换，`prompt.txt` 前置）。
