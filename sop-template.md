# AI Native 项目初始化 SOP 模板

> 把这份文档 download 到新项目根目录，按下面的步骤跑一遍，
> 就能把项目初始化成 AI Native 协作模式。
>
> 这是分享用的示例模板，请按团队实际情况调整后使用。

---

## 0. 前置条件

- [ ] 已有 git 仓库
- [ ] 已有钉钉项目群（建议一项目一群）
- [ ] 群里已配置钉钉机器人（用于自动广播）
- [ ] 每个协作人已安装 agent（Claude Code / Cursor / Codex 等）

---

## 1. 初始化项目目录结构

在项目根目录创建以下结构：

```
project-root/
├── .specs/           # 所有 spec 文档（需求边界、验收标准）
│   ├── current/      # 当前活跃 spec
│   └── archive/      # 已完成 spec 归档
├── .plans/           # agent 拆解的 plan 文档
├── .delivery/        # delivery 摘要（每次交付的完成项 + 验证结果）
├── .decisions/       # 关键决策记录（为什么这么做、考虑过什么）
├── TODO.md           # 当前待办（下一步 / 阻塞 两类）
├── AGENTS.md         # agent 协作守则（写给 agent 看的项目规则）
└── .dingtalk-bot.yml # 钉钉机器人配置（私有信息放钉钉文档，不进 git）
```

---

## 2. 统一文档模板

### spec 模板（`.specs/current/<feature>.md`）

```markdown
# <feature name>

## 背景
<为什么做这个，业务价值是什么>

## 范围
<做什么、不做什么、边界>

## 验收标准
- [ ] 验收点 1（可量化）
- [ ] 验收点 2（可量化）

## 风险
<已知风险 + 缓解措施>

## 决策记录
- <日期>：决定 X，原因是 Y
```

### delivery 模板（`.delivery/<commit-sha>.md`）

```markdown
# delivery · <feature name>

## 完成项
- <做了什么，关联 commit>

## 验证结果
- 测试通过情况：
- 验收点对照：

## 下一棒
- 谁：@<人>
- 做什么：<具体 TODO>
- 验收标准：<可量化>

## 阻塞
- <如有>
```

---

## 3. 配置钉钉机器人

### 3.1 私有信息放钉钉文档（不进 git）

把以下信息放在钉钉文档的「项目私有配置」里：

- 钉钉群机器人 webhook URL
- 钉钉机器人 secret
- API key / 服务账号
- 服务器地址 / 凭据引用

### 3.2 配置 git hook 推送到钉钉群

在 `.git/hooks/post-commit` 或 CI pipeline 里加入推送逻辑：

```bash
# 伪代码示例
commit_msg=$(git log -1 --pretty=%B)
author=$(git log -1 --pretty=%an)
delivery_summary=$(cat .delivery/$(git rev-parse HEAD).md 2>/dev/null | head -20)

curl -X POST "$DINGTALK_BOT_WEBHOOK" \
  -H "Content-Type: application/json" \
  -d "{
    \"msgtype\": \"markdown\",
    \"markdown\": {
      \"title\": \"推送 $author\",
      \"text\": \"**@$author** 提交了 $commit_msg\\n\\n$delivery_summary\"
    }
  }"
```

---

## 4. 每人配置 agent

### 4.1 安装 superpowers skill

```bash
# Claude Code 用户
claude plugin add https://github.com/obra/superpowers
```

### 4.2 接入钉钉 MCP

参考 [钉钉 MCP 文档](https://aihub.dingtalk.com/#/mcp) ，让 agent 能：
- 读写钉钉文档
- 推送群消息
- 查询群成员

### 4.3 写 AGENTS.md

在项目根目录写一份 `AGENTS.md`，告诉 agent：
- 项目背景和约束
- 哪些动作需要二次确认
- 哪些信息必须回写文档
- 群广播的触发条件

---

## 5. 建立映射关系

确保以下三件事的映射清晰且可追溯：

| 钉钉机器人 | ↔ | 项目 git 地址 | ↔ | 群里相关人 |
|---|---|---|---|---|
| bot-ai-proxy | ↔ | github.com/xxx/ai-proxy | ↔ | @jyzhao @sy.zhang @july.zhou |

把这张映射表写在 `README.md` 或钉钉文档里，新人入项目时第一眼就能看到。

---

## 6. 发布第一个任务

按闭环跑一遍，验证链路是否通：

1. 一个人写 spec → 提 PR 进 `.specs/current/`
2. 自己的 agent 读 spec，生成 plan
3. agent 执行 + 测试 + 提交
4. delivery 摘要进 `.delivery/`
5. 群机器人广播「@xxx 完成 X，下一棒 @yyy」
6. @yyy 问 agent「有什么需要我做的」，agent 列出 TODO

如果六步都跑通，初始化完成。

---

## 7. 评估指标

按以下指标跑两周，对比改造前后：

- [ ] Spec → Delivery 平均周期
- [ ] 交接等待时间
- [ ] 一次验收通过率
- [ ] 阻塞响应时间
- [ ] 关键决策回写率

---

## 8. 常见坑

- spec 写太抽象 → agent 拆 plan 跑偏
- 群机器人广播太多 → 关键通知被刷掉
- 没 delivery → 任务完成后没人知道做了什么
- 私有 token 进 git → 安全事故
- 每个人 agent 配置不一致 → 协议无法接力

---

_这份模板来自一次 AI Native 组织协作的探索分享。复制使用前请按团队实际情况调整。_
