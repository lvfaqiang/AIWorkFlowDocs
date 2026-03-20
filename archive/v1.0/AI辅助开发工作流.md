# AI 辅助开发工作流（最终版）

## Agent 角色定义

| Agent | 角色 | 职责 | 参与步骤 |
|-------|------|------|---------|
| 主 Agent | 编排者 | 沟通需求、生成 Plan、生成模块方案、任务拆分、协调各 Agent、维护知识库 | ⓪①③④⑧⑨ |
| Review Agent | 审查者 | 审查全局 Plan、审查模块实现方案 | ②⑤ |
| CodingAgent | 实现者 | 按 Todo 写代码、build 验证、commit | ⑥ |
| QA Agent | 质检者 | 模块完成后审查代码产出，对照方案验证完整性和正确性 | ⑦ |

### 主 Agent（Orchestrator）

贯穿全流程的核心角色，不直接写代码，负责协调和决策。

- 与用户沟通，理解需求
- 生成全局 Plan + Context Summary
- 确定模块列表、依赖关系、执行顺序
- 简单任务直接拆分 Todo，复杂任务委派给其他 Agent 拆分后审查
- 生成模块实现方案（文件清单、技术方案、Todo 列表）
- 接收 CodingAgent 的方案偏差反馈，更新实现方案
- 模块完成后合并知识库、更新 overview.md（兼任 Knowledge Agent 职责）
- 多人协作时负责模块分配，保证不重叠

类比：项目经理 + 技术负责人。不亲自写代码，但对全局方案和进度负责。

### Review Agent

质量审查角色，只读不写，专门挑毛病。

- 审查全局 Plan（步骤②）：需求覆盖度、技术可行性、风险识别、粒度合理性
- 审查模块实现方案（步骤⑤）：功能范围完整性、技术选型合理性、文件清单准确性、前置依赖是否满足、Todo 粒度是否合理

类比：Code Review 中的 Reviewer，但审查的是方案而不是代码。提供对抗性视角，防止主 Agent 的盲区变成执行中的坑。

### CodingAgent（Implementer）

唯一直接写代码的角色。

- 按 Todo 顺序逐个实现功能
- 严格按照模块实现方案中的文件清单和技术方案执行
- 参考指定的文件来保持代码风格一致
- 每个 Todo 完成后执行 `make build`、更新 checkbox、git commit
- 发现方案偏差时反馈主 Agent，不自作主张

类比：执行力强的开发工程师。拿到明确的需求文档和技术方案后高效实现，遇到方案问题不硬扛，及时上报。

### QA Agent

代码质量审查角色，在 CodingAgent 完成整个模块后介入。

- 对照模块实现方案，验证代码是否实现了所有功能点
- 检查边界情况是否都处理了
- 检查代码风格是否与参考文件一致
- 检查有没有硬编码、遗漏的错误处理、不合理的逻辑
- 确认 `make build` / `make test` / `make lint` 是否通过

QA 不通过 → 反馈给 CodingAgent 修复 → 修复后再次 QA，最多 2 轮，仍不通过则升级给用户。

类比：QA 工程师。不写代码，但对代码产出的质量负责。

### Agent 协作关系

```
用户
  │
  ▼
主 Agent（Orchestrator）──────────────────────────
  │           │              │              │
  │           ▼              ▼              ▼
  │     Review Agent    CodingAgent     QA Agent
  │     (审查方案)      (写代码)        (质检代码)
  │           │              │              │
  │     ┌─────┘              │         ┌────┘
  │     │ 审查反馈            │ 偏差反馈  │ 质检结果
  │     ▼                    ▼         ▼
  └──── 主 Agent 决策 ◄──────┘    通过 → 主 Agent 更新知识库
                                  不通过 → CodingAgent 修复（最多 2 轮）
```

信息流向：
- 主 Agent → Review Agent：传递 Plan / 模块方案 + Context Summary
- Review Agent → 主 Agent：返回审查意见
- 主 Agent → CodingAgent：传递当前 Todo + 模块实现方案
- CodingAgent → 主 Agent：完成通知 / 方案偏差反馈
- 主 Agent → QA Agent：模块代码完成后触发质检
- QA Agent → CodingAgent：不通过时反馈修复要求
- QA Agent → 主 Agent：通过后触发知识库更新

---

## 文档体系

```
.ai/
├── docs/
│   ├── overview.md              ← 项目总览 + 模块级进度汇总
│   ├── auth.md                  ← Auth 模块知识库文档
│   ├── order.md                 ← Order 模块知识库文档
│   └── ...
├── plans/
│   ├── auth-login.md            ← 登录功能实现方案 + Todo 列表
│   ├── auth-register.md         ← 注册功能实现方案 + Todo 列表
│   └── ...
Makefile                         ← 项目构建快捷命令
```

---

## 流程总览

```
用户需求
  │
  ▼
⓪ 项目环境检查（首次进入项目时触发）
  │
  ▼
① 主 Agent 理解需求 → 生成全局 Plan + Context Summary
  │
  ▼
② Review Agent 审查 Plan（最多 2 轮）
  │
  ▼
③ Plan 通过 → 确定模块列表与执行顺序
  │
  ▼
④ 选取下一个模块 → 生成模块实现方案（含 Todo 列表）
  │
  ▼
⑤ Review Agent 审查模块实现方案
  │
  ▼
⑥ CodingAgent 按 Todo 顺序逐个实现
  │
  ▼
⑦ QA Agent 审查代码产出（最多 2 轮）
  │
  ▼
⑧ 模块完成 → 主 Agent 验证 + 知识库更新
  │
  ▼
⑨ 回到 ④ 选取下一个模块
```

---

## 步骤详解

### ⓪ 项目环境检查（首次进入项目时触发）

每个项目首次执行工作流时，主 Agent 先检查项目环境是否满足工作流要求：

1. **检查 Makefile 是否存在**
   - 存在 → 跳过
   - 不存在 → 分析项目现有构建方式（package.json、build.gradle、Cargo.toml 等），自动生成 Makefile 包装现有构建命令

2. **检查 `.ai/` 目录结构是否完整**
   - 不存在 → 创建 `.ai/docs/` 和 `.ai/plans/` 目录
   - 部分存在 → 补全缺失的目录

3. **检查 `.ai/docs/overview.md` 是否存在**
   - 存在 → 跳过
   - 不存在 → 扫描项目结构，生成初始 overview.md（技术栈、目录结构、核心模块概览）

**好处：**
- 无论是新项目还是老项目，都能顺畅进入工作流，不会因为缺少 Makefile 或文档目录而卡住
- 老项目首次接入时自动补全基础设施，零手动配置
- 自动生成的 overview.md 为后续所有步骤提供项目全局上下文

---

### ① 主 Agent 理解需求 → 生成全局 Plan + Context Summary

主 Agent 与用户沟通，理解需求全貌，产出两样东西：

- **全局 Plan**：要做什么、分哪些模块、模块间的依赖关系、建议的执行顺序
- **Context Summary**：需求背景、技术约束、关键决策点的结构化摘要

**好处：**
- Plan 把模糊的需求变成结构化的可执行方案，防止"边做边想"导致的方向漂移
- Context Summary 是给后续所有 SubAgent 的"压缩上下文"，避免每个 SubAgent 都要重新消化原始对话，大幅节省 token

---

### ② Review Agent 审查 Plan（最多 2 轮）

Review Agent 拿到 Plan + Context Summary，从四个维度审查：

- **需求覆盖度**：Plan 是否遗漏了用户提到的需求点
- **技术可行性**：方案是否与项目技术栈兼容
- **风险识别**：哪些步骤可能出问题，有没有回退方案
- **粒度合理性**：模块划分是否太粗或太细

最多 2 轮迭代，仍有重大分歧则升级给用户决策。

**好处：**
- 引入对抗性视角，弥补主 Agent 的盲区。一个人做方案容易"自洽但有漏洞"，第二双眼睛能抓住这些
- 2 轮上限防止无限来回，保证流程推进效率
- 在执行前发现问题，修正成本极低；执行后发现问题，返工成本可能是 10 倍

---

### ③ Plan 通过 → 确定模块列表与执行顺序

根据 Plan 确定：

- **模块列表**：每个模块的职责边界
- **依赖关系**：哪些模块依赖哪些模块
- **执行顺序**：被依赖的模块先做
- **并行分配约束**：多人/多 Agent 协作时，同一模块不重复分配

简单任务主 Agent 直接拆分，复杂任务用 SubAgent 拆分后主 Agent 快速审查依赖关系、粒度和遗漏。

**好处：**
- 明确的执行顺序避免"做到一半发现前置模块没做"的阻塞
- 模块级互斥分配是多人协作下最简单有效的防冲突策略，不需要复杂的锁机制
- 剩余的少量共享文件冲突交给 git merge 手动处理即可

---

### ④ 选取下一个模块 → 生成模块实现方案

选取当前要实现的模块，产出模块实现方案，内容包括：

```markdown
## 登录模块 — 实现方案

### 功能范围
- 邮箱+密码登录
- JWT Token 签发
- 登录失败处理（用户不存在、密码错误、账号禁用）

### 技术方案
- 路由: POST /api/auth/login
- 密码比对: bcrypt
- Token: jsonwebtoken, 有效期 24h
- 响应格式: 遵循项目统一 ApiResponse 结构

### 文件清单
需要新建:
- src/api/auth/login.handler.ts
- src/api/auth/login.validator.ts

需要修改:
- src/api/routes.ts（添加路由）

需要参考（不修改）:
- src/api/auth/register.handler.ts（handler 结构参照）
- src/middleware/auth.ts（token 生成逻辑复用）
- src/models/User.ts（用户模型定义）

### 边界情况
- 用户不存在 → 返回统一错误，不暴露"用户不存在"
- 密码错误 → 同上，防止枚举攻击
- 账号禁用 → 明确提示

### Todo 列表
- [ ] T-101: 创建登录入参校验
- [ ] T-102: 实现登录 handler
- [ ] T-103: 注册路由
- [ ] T-104: 边界情况处理 + 错误码
- [ ] T-105: make build 验证
```

**好处：**
- 文件清单是这个流程的核心价值。CodingAgent 拿到方案后直接知道该读哪些文件、改哪些文件、参考哪些文件，不需要全局搜索。这一步一次性的探索成本，换来的是后续每个 Todo 实现时零探索成本，token 节省非常显著
- "需要参考"的文件明确了代码风格的来源，CodingAgent 照着写就行，不会发明新模式
- 边界情况提前列出，防止 CodingAgent 只实现 happy path
- 大多数 Todo 不需要单独的 spec 文件，模块实现方案已经覆盖了。只有极复杂的单个 Todo 才需要独立 spec

---

### ⑤ Review Agent 审查模块实现方案

Review Agent 审查模块实现方案，关注：

- 功能范围是否完整（对照全局 Plan 中对该模块的描述）
- 技术选型是否合理
- 文件清单是否准确（有没有遗漏、有没有多余）
- 依赖的前置模块是否已完成
- Todo 拆分粒度是否合理

**好处：**
- 在写代码之前再过一遍方案，成本极低但能拦住方向性错误
- 文件清单的准确性直接决定后续实现效率，值得多一双眼睛检查
- 如果前置模块没完成就开始做，会导致中途阻塞，这里提前拦住

---

### ⑥ CodingAgent 按 Todo 顺序逐个实现

CodingAgent 接收的输入：

- 当前 Todo 的描述
- 完整的模块实现方案（文件清单、技术方案、参考模式）
- 项目的 Makefile（知道怎么构建）

**执行规则：**

- 按 Todo 顺序逐个实现，每次只做一个
- 每个 Todo 完成后执行 `make build` 确保编译通过
- 每个 Todo 完成后更新模块文档中的 checkbox
- 每个 Todo 完成后 git commit，message 格式：`feat(模块): T-xxx Todo标题`
- 遇到方案偏差（比如发现参考文件的模式不适用）→ 反馈主 Agent → 主 Agent 更新方案 → 继续

**commit message 示例：**

```
feat(auth): T-101 创建登录入参校验
feat(auth): T-102 实现登录 handler
feat(auth): T-103 注册登录路由
fix(auth): T-104 边界情况处理与错误码
chore(auth): T-105 build 验证通过
```

**好处：**
- CodingAgent 拿到的是"完全预消化"的上下文，不需要自己探索代码库，实现速度快、token 消耗低
- 每个 Todo 后 `make build` 是增量验证，问题在最小范围内暴露，修复成本低。如果攒到最后才 build，一堆错误混在一起很难定位
- 每个 Todo 一个 commit，回滚粒度细——某个 Todo 有问题直接 revert 那一个 commit，不波及其他
- git log 和模块文档的 Todo 列表一一对应，跨会话恢复时即使文档状态有偏差，commit 历史不会骗人
- 方案偏差反馈机制防止 CodingAgent 自作主张偏离方案，保持整体一致性

---

### ⑦ QA Agent 审查代码产出

模块所有 Todo 完成后，QA Agent 介入审查，审查维度：

- 代码是否实现了模块实现方案中列出的所有功能点
- 边界情况是否都处理了
- 代码风格是否与参考文件一致
- 有没有硬编码、遗漏的错误处理、不合理的逻辑
- `make build` / `make test` / `make lint` 是否通过

QA 不通过 → 反馈给 CodingAgent 修复 → 修复后再次 QA，最多 2 轮，仍不通过则升级给用户。

**好处：**
- `make build` + `make test` + `make lint` 能保证编译和基本质量，但保证不了代码是否真的符合方案要求、是否有逻辑漏洞、是否遗漏边界情况
- QA Agent 对照方案做人工级别的代码审查，弥补自动化工具的盲区
- 最多 2 轮的限制防止无限修复循环，不通过则升级给用户做最终判断

---

### ⑧ 模块完成 → 主 Agent 验证 + 知识库更新

QA 通过后，主 Agent 执行：

1. 将模块实现方案中的关键信息合并到知识库模块文档（如 `.ai/docs/auth.md`）
2. 清理或归档 plan 文件
3. 更新 `overview.md` 中的模块进度

模块合并到主分支时，可根据团队偏好选择：
- 保留所有 commit（保留完整开发历史）
- squash 成一个 commit（保持主分支整洁）

**好处：**
- 知识库由主 Agent 兼任维护，省掉独立 Knowledge Agent 的调度成本
- 知识库合并保证 `.ai/docs/` 始终反映项目最新状态，不会出现文档和代码脱节
- 清理 plan 文件避免过期文档误导后续工作
- overview.md 的进度更新让全局视角始终准确

---

### ⑨ 回到 ④ 选取下一个模块

循环直到所有模块完成。

---

## Makefile 模板

新项目标配，每个项目创建时生成：

```makefile
.PHONY: build dev stg prod clean test lint

build:
	# 开发环境构建

dev:
	# 启动开发服务器

stg:
	# Staging 环境构建

prod:
	# 生产环境构建

clean:
	# 清理构建产物

test:
	# 运行测试

lint:
	# 代码检查
```

**好处：**
- 统一的构建入口，不管底层是 npm、gradle 还是 cargo，对 Agent 来说都是 `make build`
- Agent 不需要知道项目具体的构建命令，降低出错概率
- 新人（或新会话的 Agent）上手零成本

---

## 多人协作约束

唯一的规则：**任务分配时保证模块不重叠，同一模块同一时间只分配给一个 Agent。**

共享文件（routes.ts、类型定义等）的少量冲突，git merge 时手动处理。

---

## 跨会话恢复路径

```
读 overview.md → 找到当前模块和进度
  → 读模块实现方案 → 找到当前 Todo
    → 检查 git log 确认代码实际状态 → 继续工作
```

git commit 历史作为进度的"第二信源"，与文档中的 checkbox 互相印证，确保恢复准确。
