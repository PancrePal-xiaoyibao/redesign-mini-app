# 病情主动管理科普助手
## 产品需求文档 V2.0 - 公益版（2025-12-23 增强版）

---

## 1. 版本说明

本次更新（V2.0 增强版）在 V1.0 基础上新增：
- A. 完整技术栈与数据模型定义
- B. 增强分享功能（文章、AI 对话内容、成就卡片）
- C. 详细 API 设计与数据库结构
- D. 完整项目交付清单与成本预算
- E. 安全性与合规细节强化
- **F. 智能配额系统**：
  - ✓ 每日免费 10 次（日清零）
  - ✓ 完成关卡/分享文章可赚取额外配额
  - ✓ **赚取配额永久有效，永不过期，可无限累积**
  - ✓ 防止用户因"没用完"而失去已赚取奖励

---

## 2. 项目概述

### 2.1 产品名称
病情主动管理科普助手（公益小程序）

### 2.2 产品定位
面向肿瘤患者及家属的「全病程科普与风险管理公益平台」。
仅提供权威科普知识、风险识别教育及自我管理工具，不输出任何诊疗意见或处方建议。

### 2.3 目标用户
- 主要：肿瘤患者（各阶段）
- 次要：患者家属、陪护者

### 2.4 核心价值
① 降低信息壁垒  
② 强化风险意识  
③ 赋能医患沟通  
④ 整合公益资源  

### 2.5 免责声明（小程序开屏、各功能页常驻 footer）
```
⚠️ 重要声明：
本平台所有内容仅供科普与参考，不能替代医生面诊。
如有不适，请及时就医。
```

---

## 3. 整体架构图（文字描述）

```
用户端：
首页 → Roadmap → 文章/游戏/AI助手/工具/社区/新手手册

管理端：
内容管理 + 用户管理 + 工具链接管理 + 审核工作流 + API配置 + 数据分析看板
```

**技术架构：**
```
┌──────────────────────────────────────────┐
│      WeChat Mini-App (uni-app + Vue3)    │
│  ┌────────────────────────────────────┐ │
│  │ Roadmap │ 文章 │ 游戏 │ AI │ 社区 │ │
│  └────────────────────────────────────┘ │
└──────────────┬───────────────────────────┘
               │ HTTPS REST API
        ┌──────▼──────────────────┐
        │   Node.js Backend       │
        │  (Express/Koa)          │
        │ ┌──────────────────────┐│
        │ │Routes: /auth /api    ││
        │ │Services & Middleware ││
        │ └──────────────────────┘│
        └──────────┬──────────────┘
                   │
        ┌──────────┼──────────────┐
        │          │              │
    ┌───▼──┐  ┌───▼───┐  ┌──────▼──┐
    │Supa- │  │微信   │  │FastGPT/ │
    │base  │  │API    │  │Dify API │
    │(DB)  │  │       │  │         │
    └──────┘  └───────┘  └─────────┘
```

---

## 4. 功能模块（公益版）

### 4.1 指引导航图（Knowledge Roadmap）
- **可视化时间轴**：早筛 → 确诊 → 治疗 → 康复/随访
- **每节点含**：推荐文章、游戏关卡、实用工具、FAQ
- **信源标识**：A/B/C 角标直接露出于节点卡片
- **交互**：点击进入详情，支持收藏、分享

### 4.2 游戏化学习（「知识闯关」）
- **关卡内容**：仅聚焦「风险信号识别+紧急应对流程」
- **关卡难度**：分为「轻松」(1-2 星) 和「挑战」(3-5 星)
- **失败策略**：2 次机会，失败后 6h 冷却并弹出正确答案解析
- **奖励体系**：
  - 完成任何关卡 → 公益勋章+学习进度徽章
  - **完成"轻松"难度关卡 → +10 AI 配额**（每日最多 3 次，即 +30 配额）
  - 完成"挑战"难度关卡 → +20 AI 配额（每日最多 2 次，即 +40 配额）
- **分享**：完成关卡可分享成就卡片至微信，鼓励朋友参与

### 4.3 科普文章库
- **仅接入公益授权或指南开源内容**
- **每篇文章顶部展示**：信源等级、审核日期、阅读时长
- **操作**：收藏、分享、无障碍朗读
- **分享奖励**：
  - 分享文章至微信朋友圈或微信群 → **+10 AI 配额**（每日最多 3 次，即 +30 配额）
  - 分享需后台验证（通过微信分享接口回调）
- **分享详情**：
  - 分享至微信好友/朋友圈：自动生成卡片（标题+摘要+来源+二维码）
  - 分享内容预制：「我在【病情主动管理科普助手】看到一篇关于 XXX 的权威文章，推荐给你」
  - 分享链接：带 utm_source 追踪（仅统计，不用于商业）
  - 奖励弹窗：分享成功后提示「已赚取 +10 AI 配额」

### 4.4 AI 科普助手（去处方化）

#### 配额机制
- **基础每日配额**：每日免费 10 次，次日 0 点重置（仅限当天使用）
- **额外可获得配额**（永不过期，可累积）：
  - ✓ 完成"轻松"难度关卡：+10 配额（每日最多 3 次 = +30）
  - ✓ 完成"挑战"难度关卡：+20 配额（每日最多 2 次 = +40）
  - ✓ 分享文章至朋友圈/群组：+10 配额（每日最多 3 次 = +30）
  - 💡 **理论日最多可赚**：30 + 40 + 30 = **100 次/天**

#### 配额结构
- **今日免费次数**：10 次，仅限今天用，0 点清零重置
- **累积奖励次数**：通过任务赚取的配额，**永久有效且可无限累积**，不受日期限制
- **使用顺序**：优先使用当日免费 10 次，用完后再扣除累积奖励配额

#### 配额显示
- 显示详细配额：
  ```
  今日免费：10/10 次（今晚 24:00 清零）
  累积奖励：47 次（永久有效）
  ─────────────────────
  总计可用：57 次
  ```
- 配额快用完时提示「分享文章或完成关卡获得更多配额」
- **系统 prompt**：内置 42 条拒答模板
- **所有回复文末**：固定免责声明
- **上传报告**：仅 OCR 提取数值+科普解释，30 天原文粉碎
- **分享功能**：
  - 用户可分享单条 AI 回复至微信
  - 分享文案预制：「AI 科普助手回答：[回复内容摘要]」
  - 自动脱敏：不包含用户隐私、对话 ID、时间戳
  - 分享链接：链接可跳转至对话详情（已登录用户可查看完整对话）
  - ⚠️ 注意：分享 AI 回复本身不额外赠送配额（仅统计分享率）

### 4.5 实用工具跳转（公益外链）
- **工具类别**：Knows小程序，腾讯药典小程序，丁香园药典小程序，临床查询小程序，症状日记、用药提醒、营养计算器、医院科室查询
- **统一 web-view 套壳**，顶部返回条强制保留
- **合作方备案**：仅接入经医学顾问委员会审核的公益合作方

### 4.6 社区交流（「我们讨论」- BBS 轻社交）

#### 4.6.1 功能概述
**社区定位**：轻社交内容平台，允许患者/家属分享就诊经验、心理支持、生活经验，但严格禁止医学诊疗讨论。

#### 4.6.2 社区用户目标与价值

**主要使用者**：
- 肿瘤患者：分享诊疗流程、生活经验、心理调适
- 患者家属：交流陪护经验、心理支持、就业平衡
- 康复患者：分享康复经验、鼓励新患者

**社区核心价值**：
- 降低患者孤独感，建立同伴支持网络
- 聚合真实患者经验（非医学诊断）
- 强化患者信心和自我管理意识
- 识别心理危机，及时提供帮助

#### 4.6.3 发布内容规范

**允许发布的内容**：
- ✅ 就诊经验分享（医院选择、检查流程、预约技巧、医生沟通）
- ✅ 心理支持话题（心理调整、家庭关系、工作平衡、情绪管理）
- ✅ 生活经验建议（营养调理、运动康复、日常护理、防止副作用）
- ✅ 勋章/成就展示（完成关卡、学习进度、坚持打卡）
- ✅ 患者故事（诊断过程、治疗心路、康复历程、鼓励他人）
- ✅ 医学科普讨论（基于权威指南的问答、知识补充）

**禁止发布的内容**（系统自动过滤 + 人工审核）：
- ❌ 诊断建议（「你可能是...」「建议你做...检查」「应该进行...治疗」）
- ❌ 用药讨论（具体药物名称、剂量建议、用法频率、药物效果对比）
- ❌ 治疗方案建议（「我用的疗法很有效...」「强烈推荐...方案」）
- ❌ 商业推广（产品代理、私域导流、链接宣传、微商推广）
- ❌ 人身攻击（辱骂、骚扰、隐私泄露、实名曝光）
- ❌ 医疗欺诈（虚假案例、偏方秘方、神奇疗效宣传）

#### 4.6.4 用户身份与等级体系

**用户身份标签**：
- 🟦 **患者**：已通过身份认证的确诊患者（可选填癌症类型）
- 🟨 **家属**：患者的陪护者/家属
- 🟩 **医生**（可选）：合作医院的医学顾问，可进行科学回复（标记为「医学建议」）
- 👑 **版主**：社区审核员，可对不当内容隐藏/删除

**用户等级系统**（Level 1-10）：
```
Level 1-3：新手（灰色）
  • 需人工审核首条发言
  • 发帖上限：5 帖/日
  • 不可直接评论医学建议

Level 4-7：活跃（绿色）
  • 发帖可先发后审（提高发布速度）
  • 发帖上限：10 帖/日
  • 可评论所有帖子
  • 点赞/评论数更重

Level 8-10：社区贡献者（金色 VIP）
  • 发帖完全不审（信任发布）
  • 无发帖上限
  • 可帮助版主进行反垃圾举报
  • 标记为「社区贡献者」，获得特殊徽章

等级提升条件：
- 发帖被赞 ≥ 100 次 → +1 Level
- 发帖被评论 ≥ 50 次 → +1 Level
- 连续 7 天无违规 → +1 Level
- 被其他用户举报 / 内容被删除 → -1 Level
```

#### 4.6.5 社区审核工作流

**审核策略**：
- **Level 1-3**：所有帖子 `先审后发`（用户发布后需等待审核）
- **Level 4-7**：`先发后审`（可立即显示，但后台监控）
- **Level 8-10**：不审（信任发布）

**审核周期**：
- SLA 承诺：4 小时内审核回复（工作日）
- 周末/夜间：次日 9 点前完成

**审核结果通知**：
```
✓ 已通过  → 用户可见该帖，通知：「您的帖子已发布」
⏳ 待审核 → 用户可见「待审核」标签，其他用户不可见
✗ 已拒绝 → 用户收到私信，说明原因和申诉入口
```

**拒绝原因分类**：
- 违规类型 A：医学诊疗讨论 → 建议「查看科普资源」
- 违规类型 B：商业推广 → 建议「使用工具中心」
- 违规类型 C：人身攻击 → 警告并扣 1 Level
- 违规类型 D：其他 → 说明具体原因

#### 4.6.6 社区帖子结构

**帖子包含字段**：
```
• 发布者信息
  - 用户头像（32px）
  - 昵称（可匿名发布）
  - 身份标签（患者/家属/医生）
  - 等级徽章（Level 1-10）
  
• 帖子内容
  - 标题（10-100 字，必填）
  - 正文（20-2000 字，必填）
  - 配图（可选，最多 4 张）
  - 标签（可选，最多 3 个预设标签）
  
• 审核信息
  - 审核状态（✓已通过 / ⏳待审核 / ✗已拒绝）
  - 发布时间
  - 更新时间
  
• 交互数据
  - 评论数
  - 点赞数 (❤️)
  - 浏览数 (👁️)
  - 分享数 (📤)
  - 举报数（仅版主可见）
```

**预设标签类别**：
- 「就诊体验」：关于医院/医生的经验
- 「心理支持」：心理调适、家庭关系
- 「生活经验」：营养、运动、日常护理
- 「勋章展示」：学习成就分享
- 「患者故事」：个人诊疗历程
- 「科普讨论」：基于权威内容的讨论

#### 4.6.7 评论系统

**评论规范**：
```
• 一级评论：用户对帖子的直接回复（最多 500 字）
• 二级评论：用户对一级评论的回复（最多 300 字）
• 不支持三级及以上嵌套（防止过度分散讨论）
• 超过 5 条二级评论自动折叠「更多回复」

评论审核：
- 与帖子同等级策略（先审/后审/不审）
- 若回复内容违规，该评论被隐藏
- 用户可申诉被隐藏的评论

互动操作：
- 点赞评论（可多次，但仅计 1 赞）
- 举报评论（涉及违规、骚扰）
- 回复评论（@提及评论者）
```

#### 4.6.8 安全与危机预警

**AI 自动检测**：
- 检测医学诊疗关键词（如「你可能是」「建议你做」）
- 检测心理危机关键词（「自杀」「放弃」「活不下去」等）
- 检测商业推广关键词（「代理」「招商」「私信」等）

**危机预警处理**：
- 系统检测到危机关键词后，自动触发关键弹窗
- 用户可点击「心理危机救助」按钮（社区页面顶部固定）直接调用：
  - 拨打心理援助热线（全国 24h：400-161-9995）
  - 联系合作医院心理医生
  - 查看心理资源库

**危机弹窗示例**：
```
⚠️ 我们关心您的身心健康
如果您正经历心理困扰或危机，请立即寻求专业帮助：

[☎️ 拨打心理援助热线]  [👨‍⚕️ 联系医生]  [📚 查看资源]

同时通知：审核员、医学顾问委员会
帮助信息会被加密存储，仅用于危机干预
```

**一键举报机制**：
- 用户可举报任何不当帖子/评论
- 举报时需选择原因（诊疗讨论/商业推广/骚扰等）
- 版主后台收到举报队列，优先处理

**内容风险分级**：
| 风险等级 | 特征 | 处理方式 |
|--------|------|--------|
| 低风险 | 表述不当但无害 | 建议编辑，可发布 |
| 中风险 | 涉及诊疗讨论 | 驳回，告知原因 |
| 高风险 | 商业推广、骚扰 | 删除，扣 Level |
| 极高风险 | 心理危机、犯法 | 删除，通知用户+版主 |

#### 4.6.9 社区分享奖励

**分享奖励规则**：
```
用户分享自己的帖子至微信朋友圈/群 → 需后台验证
✓ 分享成功验证 → +5 AI 配额（每日最多 2 次 = +10）
  
验证方式：
- 用户点击「分享」按钮
- 弹窗显示分享卡片（包含二维码、标题、摘要）
- 分享 72h 内检查链接点击，确认分享有效
- 自动赠送配额

分享卡片内容：
「我在【病情主动管理科普助手】社区分享了一则经验：
  [帖子标题]
  
来自：[用户昵称]（Level [等级]）
点赞：[点赞数]  评论：[评论数]

点击查看完整讨论 →」
```

#### 4.6.10 社区数据统计

**社区运营看板**（仅版主可见）：
```
• 日发帖量、日评论量、日举报数
• 审核队列深度（待审帖子数）
• 平均审核耗时
• 违规内容分布（诊疗/推广/骚扰等）
• 心理危机关键词命中次数（仅统计，不留原文）
• 高活跃用户排行
• 被举报最多的用户
```

#### 4.6.11 社区禁言与处罚

**进阶违规处理**：
```
首次违规：删除内容 + 警告信息 + 扣 1 Level
二次违规：删除内容 + 扣 3 Level + 3 天禁言
三次违规：删除所有内容 + 扣 5 Level + 7 天禁言
多次严重违规：永久禁言（内容保留但不可交互）

禁言期间用户可申诉，版主 24h 内回复
```

#### 4.6.12 分享功能

**帖子分享**：
- 用户可分享自己发布的帖子至微信好友/朋友圈
- 分享文案预制：「我在社区分享了关于 XXX 的经验，欢迎讨论」
- 分享链接带 utm_source 追踪（仅统计，不用于商业）
- 分享验证成功后 +5 AI 配额（每日最多 2 次）

**评论分享**（仅高质量评论）：
- 可分享他人的优质评论（赞数 ≥ 20）
- 自动添加「推荐评论」标记
- 分享者不获得配额，但评论者可获得「被分享」徽章

### 4.7 新手手册（H5）
- **术语速查**、**就诊流程图解**、**如何与医生沟通模板**、**紧急情况 SOP**
- **支持离线阅读**（缓存至本地）

---

## 5. 数据模型（概要）

### 5.1 核心表结构

**users 表：**
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  openid VARCHAR(255) UNIQUE NOT NULL,  -- 微信 OpenID
  identity_tag VARCHAR(50),  -- 'patient' | 'family'
  username VARCHAR(255),
  phone VARCHAR(20) ENCRYPTED,  -- 加密存储
  cancer_type VARCHAR(100),
  learning_progress INT DEFAULT 0,  -- 完成百分比 0-100
  
  -- AI 配额管理（分离日配额和累积奖励）
  ai_daily_quota INT DEFAULT 10,  -- 今日剩余免费次数（0 点重置）
  ai_accumulated_quota INT DEFAULT 0,  -- 累积奖励配额（通过任务赚取，永久有效）
  ai_daily_reset_at TIMESTAMP,  -- 下次日配额重置时间
  
  -- 今日任务计数（用于限制日奖励上限）
  games_easy_completed_today INT DEFAULT 0,  -- 今日完成 easy 关卡数（限 3 次）
  games_hard_completed_today INT DEFAULT 0,  -- 今日完成 hard 关卡数（限 2 次）
  articles_shared_today INT DEFAULT 0,  -- 今日分享文章数（限 3 次）
  task_reset_at TIMESTAMP,  -- 任务计数重置时间（次日 0 点）
  
  badges TEXT[] DEFAULT '{}',  -- 已获勋章 ID 数组
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);
```

**articles 表：**
```sql
CREATE TABLE articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(500) NOT NULL,
  content TEXT NOT NULL,
  summary VARCHAR(1000),
  source_level VARCHAR(1) CHECK (source_level IN ('A', 'B', 'C')),
  category VARCHAR(100),  -- Roadmap 阶段标签
  author_name VARCHAR(255),
  reviewer_id UUID REFERENCES users(id),  -- 审核人
  review_status VARCHAR(50),  -- 'pending' | 'approved' | 'rejected'
  reviewed_at TIMESTAMP,
  is_published BOOLEAN DEFAULT false,
  published_at TIMESTAMP,
  view_count INT DEFAULT 0,
  share_count INT DEFAULT 0,  -- 分享计数
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_articles_source_level ON articles(source_level);
CREATE INDEX idx_articles_status ON articles(review_status, is_published);
```

**game_levels 表：**
```sql
CREATE TABLE game_levels (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(255) NOT NULL,
  roadmap_stage VARCHAR(100),  -- 对应 Roadmap 阶段
  question TEXT NOT NULL,
  correct_answer VARCHAR(255) NOT NULL,
  options TEXT[] NOT NULL,  -- 选项数组
  explanation TEXT,  -- 答案解析
  source_level VARCHAR(1),
  difficulty INT DEFAULT 1,  -- 1-5 难度
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW()
);
```

**ai_conversations 表：**
```sql
CREATE TABLE ai_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  question TEXT NOT NULL,
  response TEXT NOT NULL,
  rejection_reason VARCHAR(255),  -- 如果被拒答，记录原因
  is_rejected BOOLEAN DEFAULT false,
  user_feedback VARCHAR(10),  -- 'thumbs_up' | 'thumbs_down'
  share_count INT DEFAULT 0,  -- 分享计数
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_ai_conversations_user ON ai_conversations(user_id);
CREATE INDEX idx_ai_conversations_status ON ai_conversations(is_rejected, created_at DESC);
```

**community_posts 表：**
```sql
CREATE TABLE community_posts (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(100) NOT NULL,  -- 帖子标题
  content TEXT NOT NULL,  -- 帖子正文
  summary VARCHAR(500),  -- 摘要（用于列表展示）
  images TEXT[],  -- 图片 URL 数组（最多 4 张）
  tags TEXT[] DEFAULT '{}',  -- 标签数组（预设标签）
  audit_status VARCHAR(50) DEFAULT 'pending',  -- 'pending' | 'approved' | 'rejected'
  reject_reason VARCHAR(255),  -- 拒绝原因
  auditor_id UUID REFERENCES users(id),
  reviewed_at TIMESTAMP,
  
  -- 统计信息
  like_count INT DEFAULT 0,
  comment_count INT DEFAULT 0,
  view_count INT DEFAULT 0,
  share_count INT DEFAULT 0,
  report_count INT DEFAULT 0,
  
  -- 其他
  is_pinned BOOLEAN DEFAULT false,  -- 版主精选
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP  -- 软删除
);

CREATE INDEX idx_community_posts_status ON community_posts(audit_status, created_at DESC);
CREATE INDEX idx_community_posts_user ON community_posts(user_id, created_at DESC);
CREATE INDEX idx_community_posts_reports ON community_posts(report_count DESC);
```

**community_comments 表：**
```sql
CREATE TABLE community_comments (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  post_id UUID NOT NULL REFERENCES community_posts(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  parent_comment_id UUID REFERENCES community_comments(id) ON DELETE CASCADE,  -- 二级回复
  content TEXT NOT NULL,  -- 评论内容（最多 500 字）
  
  audit_status VARCHAR(50) DEFAULT 'approved',  -- 同帖子审核策略
  
  -- 统计信息
  like_count INT DEFAULT 0,
  reply_count INT DEFAULT 0,
  report_count INT DEFAULT 0,
  
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP
);

CREATE INDEX idx_community_comments_post ON community_comments(post_id, created_at DESC);
CREATE INDEX idx_community_comments_user ON community_comments(user_id, created_at DESC);
CREATE INDEX idx_community_comments_parent ON community_comments(parent_comment_id);
```

**community_likes 表：**
```sql
CREATE TABLE community_likes (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID REFERENCES community_posts(id) ON DELETE CASCADE,  -- 可为空
  comment_id UUID REFERENCES community_comments(id) ON DELETE CASCADE,  -- 可为空
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, post_id),  -- 用户对帖子仅可点赞 1 次
  UNIQUE(user_id, comment_id)
);
```

**community_reports 表：**
```sql
CREATE TABLE community_reports (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  reporter_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID REFERENCES community_posts(id) ON DELETE CASCADE,  -- 可为空
  comment_id UUID REFERENCES community_comments(id) ON DELETE CASCADE,  -- 可为空
  reason VARCHAR(50) NOT NULL,  -- 'diagnosis' | 'commercial' | 'harassment' | 'other'
  description TEXT,
  status VARCHAR(50) DEFAULT 'pending',  -- 'pending' | 'reviewed' | 'resolved'
  reviewed_by UUID REFERENCES users(id),
  reviewed_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_community_reports_status ON community_reports(status, created_at DESC);
CREATE INDEX idx_community_reports_post ON community_reports(post_id);
```

**user_level 表：**
```sql
CREATE TABLE user_level (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL UNIQUE REFERENCES users(id) ON DELETE CASCADE,
  community_level INT DEFAULT 1,  -- Level 1-10
  post_count INT DEFAULT 0,  -- 发帖数
  like_received INT DEFAULT 0,  -- 获赞数
  comment_count INT DEFAULT 0,  -- 评论数
  violation_count INT DEFAULT 0,  -- 违规次数
  is_banned BOOLEAN DEFAULT false,  -- 是否被禁言
  ban_until TIMESTAMP,  -- 禁言到期时间
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_user_level_community ON user_level(community_level DESC);
```

**community_shares 表（用于记录分享奖励）：**
```sql
CREATE TABLE community_shares (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  post_id UUID NOT NULL REFERENCES community_posts(id) ON DELETE CASCADE,
  share_channel VARCHAR(50),  -- 'moments' | 'group' | 'friend'
  quota_rewarded INT DEFAULT 5,  -- 赠送的配额数
  share_verified BOOLEAN DEFAULT false,  -- 微信分享是否已验证
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_community_shares_user_date ON community_shares(user_id, created_at DESC);
```

**saved_articles 表：**
```sql
CREATE TABLE saved_articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  article_id UUID NOT NULL REFERENCES articles(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, article_id)
);
```

**article_shares 表（用于记录分享奖励）：**
```sql
CREATE TABLE article_shares (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  article_id UUID NOT NULL REFERENCES articles(id) ON DELETE CASCADE,
  share_channel VARCHAR(50),  -- 'moments' | 'group' | 'friend'
  quota_rewarded INT DEFAULT 10,  -- 赠送的配额数
  share_verified BOOLEAN DEFAULT false,  -- 微信分享是否已验证
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_article_shares_user_date ON article_shares(user_id, created_at DESC);
CREATE INDEX idx_article_shares_verified ON article_shares(share_verified, created_at DESC);
```

**game_quota_rewards 表（用于记录关卡奖励）：**
```sql
CREATE TABLE game_quota_rewards (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  level_id UUID NOT NULL REFERENCES game_levels(id) ON DELETE CASCADE,
  difficulty VARCHAR(50),  -- 'easy' (1-2 星) | 'hard' (3-5 星)
  quota_rewarded INT,  -- easy: 10, hard: 20
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_game_quota_rewards_user_date ON game_quota_rewards(user_id, created_at DESC);
```

**game_progress 表：**
```sql
CREATE TABLE game_progress (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  level_id UUID NOT NULL REFERENCES game_levels(id) ON DELETE CASCADE,
  attempts INT DEFAULT 0,
  passed BOOLEAN DEFAULT false,
  cooldown_until TIMESTAMP,  -- 失败冷却时间
  passed_at TIMESTAMP,
  UNIQUE(user_id, level_id)
);
```

---

## 6. 用户端旅程简表（MVP）

```
注册 → 微信授权 → 选身份（患者/家属）
  ↓
首次进入 Roadmap
  ↓
推荐第一篇权威文章（A 级）
  ↓
完成首个游戏（获勋章）
  ↓
解锁 AI 10 次配额
  ↓
邀请分享（好友点击链接注册 +2 配额）
```

---

## 7. 后端管理模块

### 7.1 用户管理
- **字段**：openid、身份标签、学习进度、AI 当日已用次数、勋章列表
- **数据脱敏**：手机号加密存储，仅公益运营可见
- **搜索**：按注册日期、活跃度、身份筛选
- **操作**：禁用账号、重置配额、发送系统消息

### 7.2 内容管理
- **文章/关卡/FAQ**：支持信源等级下拉选择（A/B/C）
- **审核流**：
  - C 级 → 医学审核员 1 人签字
  - A/B 级 → 双人双签（医学顾问委员会成员）
- **操作**：编辑、发布/下线、版本历史、审核评论
- **发布即生成**：静态 CDN，支持一键下线

### 7.3 工具链接管理
- **增删改外链**：名称、图标、排序、合作方备案号
- **字段**：链接、公益合作方备案号、上线状态、分类
- **验证**：合作方必须通过医学顾问委员会审核

### 7.4 审核工作流
- **内容审核队列**：显示待审核文章、帖子、评论
- **自动拒答日志**：小时级同步，8h 内人工复核
- **用户反馈**：用户可对 AI 回复点「踩」，连踩 3 次自动提交人工复核
- **社区先审后发**：展示待审帖子，可批量审核

### 7.5 数据分析看板（完整版）

#### 概要分析模块：

**① 活跃概览**
- DAU、WAU、MAU
- 人均日访问时长、人均启动次数
- 新增注册用户数（日/周/月）

**② 内容使用**
- Roadmap 各阶段节点点击率 Top10
- 文章阅读完成率（滑动到文末即算完成）
- 文章分享率排行
- 游戏关卡通过率 & 易错题分布
- 游戏平均通过次数

**③ AI 使用**
- 日提问总量、拒答率、用户评分（thumbs-up/down）
- 高频关键词词云（脱敏）
- AI 分享率排行

**④ 社区健康度**
- 日发帖量、举报率、先审后发平均耗时
- 心理危机关键词命中次数（仅统计，不留原文）
- 社区帖子分享率

**⑤ 留存与转化**
- 次日/7日/30日留存
- 新手任务完成率（注册 → 首次阅读 → 首次游戏 → 获得勋章）
- 身份维度下钻（患者 vs 家属）
- AI 配额使用率

**可视化**：日趋势折线、环形占比、地图热力（省份活跃分布）  
**导出**：支持 CSV/Excel，仅公益项目组权限，日志记录

### 7.6 系统监控
- 小程序码包版本管理
- 接口 5xx 报警
- AI 调用余量预警
- 数据库性能监控
- 用户反馈队列

---

## 8. 合规与安全（重点）

### 8.1 内容审核流程

| 等级 | 定义 | 审核人数 | 审核周期 |
|------|------|--------|--------|
| **A 级** | 国际/国家权威指南原文（NCCN、ESMO、CSCO、卫健委诊疗规范） | 2 人（医学顾问委员会） | 24h |
| **B 级** | 三甲肿瘤科主任署名文章，已正式发表于核心期刊/公众号 | 1 人 + 交叉验证 | 12h |
| **C 级** | 平台自产，经医学审核员双人复核 | 2 人 | 8h |

**显示**：文章/关卡/AI 回复均在左上角露标「A」「B」「C」

### 8.2 AI 输出过滤
- **双层门禁**：模型层（system prompt 42 条拒答模板）+ 业务层（关键词过滤）
- **拒答日志**：小时级同步，8h 内人工复核
- **用户反馈**：用户可对 AI 回复点「踩」，连踩 3 次自动提交人工复核
- **每日监控**：拒答率不超过 2%，超过则告警

### 8.3 数据隐私
- **仅收集业务最小化字段**：openid、身份、癌症类型（可选）
- **上传报告**：30 天自动粉碎，用户可一键提前删除
- **服务端日志**：90 天脱敏滚动删除
- **分享操作**：分享链接不含用户 ID、隐私数据
- **用户权利**：
  - 数据导出（CSV 格式）
  - 数据删除（软删除，30 天后物理删除）
  - 修正权（可修改个人信息）

### 8.4 安全措施

```typescript
// 1. 密码加密（如有邮箱注册）
import bcryptjs from 'bcryptjs';
const hashedPassword = await bcryptjs.hash(password, 10);

// 2. JWT Token (有效期短)
const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '1h' }  // 1 小时过期，需要刷新
);

// 3. HTTPS 强制（环境变量 + 代理配置）

// 4. CORS 限制
const cors = {
  origin: process.env.ALLOWED_ORIGINS.split(','),
  credentials: true,
  methods: ['GET', 'POST', 'PUT', 'DELETE']
};

// 5. 请求速率限制
import rateLimit from 'express-rate-limit';
const limiter = rateLimit({
  windowMs: 15 * 60 * 1000,  // 15 minutes
  max: 100  // 100 requests per window
});

// 6. 敏感字段加密
import crypto from 'crypto';
const encryptPhone = (phone) => {
  return crypto.createCipher('aes-256-cbc', process.env.ENCRYPT_KEY).update(phone, 'utf8', 'hex');
};
```

### 8.5 引导就医
- **危险信号关键词**：任何涉及的页面，固定悬浮按钮「立即联系医生」
- **功能**：跳转至手机拨号盘（号码读取用户本地通讯录，平台不预设）
- **例子**：「胸痛」「咳血」「高烧不退」等均触发该按钮

---

## 9. API 设计概览

### 9.1 认证模块 `/api/auth`

#### 微信授权登录
```
POST /api/auth/wechat-login
{
  "code": "微信授权 code",
  "iv": "初始向量",
  "encryptedData": "加密数据"
}

Response (200):
{
  "success": true,
  "data": {
    "access_token": "jwt_token",
    "refresh_token": "refresh_token",
    "user": {
      "id": "uuid",
      "openid": "wx_xxx",
      "identity_tag": null  // 需要补全
    }
  }
}
```

#### 选择身份
```
POST /api/auth/set-identity
Headers: { "Authorization": "Bearer token" }
{
  "identity_tag": "patient" | "family",
  "cancer_type": "lung" | "breast" | ...  // 可选
}

Response (200):
{ "success": true }
```

#### 登出
```
POST /api/auth/logout
Headers: { "Authorization": "Bearer token" }

Response (200):
{ "success": true }
```

### 9.2 内容模块 `/api/content`

#### 获取 Roadmap
```
GET /api/content/roadmap

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "stage": "early_screening",  // 早筛/确诊/治疗/康复
      "label": "早期筛查",
      "articles": [{ "id", "title", "source_level" }],
      "levels": [{ "id", "title", "difficulty" }],
      "faqs": [{ "id", "question" }]
    }
  ]
}
```

#### 获取文章列表
```
GET /api/content/articles?source_level=A,B,C&category=science&page=1&limit=10

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "...",
      "summary": "...",
      "source_level": "A",
      "view_count": 234,
      "share_count": 45,
      "is_saved": false,
      "published_at": "2025-01-15T10:00:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 10, "total": 100 }
}
```

#### 获取文章详情
```
GET /api/content/articles/:id

Response (200):
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "...",
    "content": "...",
    "source_level": "A",
    "author_name": "...",
    "reviewed_at": "...",
    "view_count": 234,
    "share_count": 45,
    "is_saved": false,
    "read_time": "5 分钟"
  }
}
```

#### 收藏文章
```
POST /api/content/articles/:id/save
Headers: { "Authorization": "Bearer token" }

Response (201):
{ "success": true }
```

#### 分享文章
```
POST /api/content/articles/:id/share
Headers: { "Authorization": "Bearer token" }
{ 
  "channel": "moments" | "group"  -- 朋友圈或群组
}

Response (200):
{
  "success": true,
  "data": {
    "share_url": "https://app.example.com/articles/uuid?utm_source=wechat",
    "title": "文章标题",
    "description": "文章摘要",
    "quota_earned": 10,  -- 新增：本次分享赚取的配额
    "remaining_quota_today": 30,  -- 新增：今日剩余可分享次数（最多 3 次）
    "message": "✓ 已赚取 +10 AI 配额"
  }
}
```

**注意**：
- 需通过微信 SDK 验证分享是否真实完成（通过 `wx.onMenuShareAppMessage` 回调）
- 每日最多分享 3 篇文章获得奖励
- 同一文章同一用户当日只能获得一次分享奖励

### 9.3 游戏模块 `/api/game`

#### 获取关卡列表
```
GET /api/game/levels?stage=early_screening&page=1

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "关卡标题",
      "difficulty": 2,
      "passed": false,
      "attempts": 0,
      "cooldown_until": null
    }
  ]
}
```

#### 获取关卡详情
```
GET /api/game/levels/:id
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "id": "uuid",
    "question": "...",
    "options": ["选项1", "选项2", "选项3"],
    "source_level": "B",
    "difficulty": 2
  }
}
```

#### 提交答案
```
POST /api/game/levels/:id/answer
Headers: { "Authorization": "Bearer token" }
{
  "selected_option": "选项1"
}

Response (200):
{
  "success": true,
  "data": {
    "correct": true | false,
    "correct_answer": "正确选项",
    "explanation": "答案解析",
    "badge_earned": "勋章ID",  // 如果通过
    "quota_rewarded": 0,  // 新增：本次关卡赚取的 AI 配额
    "difficulty": "easy" | "hard",
    "quota_message": "✓ 已赚取 +10 AI 配额"  // 如果通过且未超日限
  }
}

Response (200) - 如果未通过：
{
  "success": false,
  "data": {
    "correct": false,
    "attempts_remaining": 1,
    "cooldown_minutes": 360  // 失败需冷却 6 小时
  }
}
```

**配额奖励规则**：
- ✓ 通过 "轻松" 难度（1-2 星）：+10 AI 配额，每日最多 3 次
- ✓ 通过 "挑战" 难度（3-5 星）：+20 AI 配额，每日最多 2 次
- ❌ 失败或超出日限：不赠送配额

#### 分享成就
```
POST /api/game/levels/:id/share-achievement
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "share_url": "https://app.example.com/achievements/uuid",
    "title": "我完成了知识闯关关卡",
    "image": "achievement_card.png"
  }
}
```

### 9.4 AI 模块 `/api/ai`

#### 获取配额
```
GET /api/ai/quota
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "daily_quota": 10,
    "used_today": 3,
    "remaining": 7,
    "reset_at": "2025-01-16T00:00:00Z"
  }
}
```

#### 提问
```
POST /api/ai/chat
Headers: { "Authorization": "Bearer token" }
{
  "question": "肺癌早期症状有哪些？",
  "report_image": null  // 可选：上传报告
}

Response (200):
{
  "success": true,
  "data": {
    "conversation_id": "uuid",
    "question": "...",
    "response": "...",
    "is_rejected": false,
    "rejection_reason": null,
    "created_at": "2025-01-15T10:00:00Z"
  }
}
```

#### 分享 AI 回复
```
POST /api/ai/chat/:id/share
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "share_url": "https://app.example.com/ai-responses/uuid",
    "content": "AI 回复摘要（脱敏）",
    "disclaimer": "免责声明"
  }
}
```

#### 反馈 AI 回复
```
POST /api/ai/chat/:id/feedback
Headers: { "Authorization": "Bearer token" }
{
  "feedback": "thumbs_up" | "thumbs_down"
}

Response (200):
{ "success": true }
```

### 9.5 社区模块 `/api/community`

#### 获取帖子列表
```
GET /api/community/posts?page=1&limit=20&sort=latest | popular

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "username": "匿名用户",  // 脱敏
      "content": "...",
      "share_count": 12,
      "report_count": 0,
      "is_pinned": false,
      "created_at": "2025-01-15T10:00:00Z"
    }
  ],
  "pagination": { "page": 1, "limit": 20, "total": 500 }
}
```

#### 发布帖子
```
POST /api/community/posts
Headers: { "Authorization": "Bearer token" }
{
  "content": "我的就诊经历..."
}

Response (201):
{
  "success": true,
  "data": {
    "id": "uuid",
    "status": "pending"  // 先审后发
  }
}
```

#### 举报帖子
```
POST /api/community/posts/:id/report
Headers: { "Authorization": "Bearer token" }
{
  "reason": "色情" | "暴力" | "医疗建议" | "其他",
  "details": "详情说明"
}

Response (201):
{ "success": true }
```

#### 分享帖子
```
POST /api/community/posts/:id/share
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "share_url": "https://app.example.com/community/posts/uuid",
    "title": "社区精彩讨论"
  }
}
```

### 9.6 用户模块 `/api/user`

#### 获取用户信息
```
GET /api/user/profile
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "id": "uuid",
    "openid": "wx_xxx",
    "identity_tag": "patient",
    "cancer_type": "lung",
    "learning_progress": 45,  // 百分比
    "ai_quota": {
      "daily": 10,
      "used_today": 3,
      "remaining": 7
    },
    "badges": ["badge_1", "badge_2"]
  }
}
```

#### 获取学习进度
```
GET /api/user/progress
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "roadmap_progress": {
      "early_screening": 80,  // 完成百分比
      "diagnosis": 60,
      "treatment": 30,
      "recovery": 0
    },
    "articles_read": 15,
    "games_passed": 8,
    "badges_earned": 5,
    "ai_usage": { "daily": 10, "used": 3 }
  }
}
```

#### 获取勋章
```
GET /api/user/badges
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "badge_1",
      "name": "科普小卫士",
      "description": "阅读 10 篇文章",
      "icon": "badge_1.png",
      "earned_at": "2025-01-15T10:00:00Z"
    }
  ]
}
```

#### 获取收藏文章
```
GET /api/user/saved-articles?page=1&limit=10
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "...",
      "summary": "...",
      "source_level": "A",
      "saved_at": "2025-01-15T10:00:00Z"
    }
  ]
}
```

#### 导出个人数据
```
GET /api/user/export
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "download_url": "https://cdn.example.com/exports/user_uuid_data.csv",
    "expires_at": "2025-01-22T10:00:00Z"
  }
}
```

#### 删除账号
```
POST /api/user/delete
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "message": "账号将在 30 天后删除，期间可恢复"
}
```

### 9.7 管理后台模块 `/api/admin`

#### 获取数据看板
```
GET /api/admin/dashboard
Headers: { "Authorization": "Bearer admin_token" }

Response (200):
{
  "success": true,
  "data": {
    "active_users": { "dau": 1500, "wau": 8000, "mau": 25000 },
    "content": {
      "articles_published": 150,
      "articles_pending": 12,
      "avg_completion_rate": 0.72
    },
    "ai": {
      "daily_questions": 5000,
      "rejection_rate": 0.015,
      "avg_rating": 4.2
    },
    "community": {
      "daily_posts": 200,
      "report_rate": 0.008,
      "avg_audit_time": "2.5h"
    },
    "retention": {
      "day1": 0.65,
      "day7": 0.35,
      "day30": 0.20
    }
  }
}
```

#### 审核内容
```
POST /api/admin/content/:id/review
Headers: { "Authorization": "Bearer admin_token" }
{
  "status": "approved" | "rejected",
  "comment": "审核意见"
}

Response (200):
{ "success": true }
```

#### 审核社区帖子
```
POST /api/admin/community/posts/:id/review
Headers: { "Authorization": "Bearer admin_token" }
{
  "status": "approved" | "rejected",
  "reason": "理由"
}

Response (200):
{ "success": true }
```

#### 查看 AI 拒答日志
```
GET /api/admin/ai/rejections?page=1&limit=50

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "user_id": "uuid",
      "question": "...",
      "rejection_reason": "处方检测",
      "created_at": "2025-01-15T10:00:00Z",
      "reviewed": false
    }
  ]
}
```

---

## 10. 技术栈详情

### 10.1 前端技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **框架** | uni-app + Vue 3 | 跨平台小程序框架 |
| **状态管理** | Pinia | 轻量级状态管理 |
| **网络请求** | axios | HTTP 客户端 |
| **本地存储** | uni-app Storage API | 跨平台存储 |
| **日期处理** | dayjs | 轻量级日期库 |
| **UI 框架** | uni-ui + 自定义 | 组件库 + 品牌定制 |
| **无障碍** | WAI-ARIA | 屏幕阅读器支持 |
| **语言** | TypeScript | 类型安全 |

**前端目录结构：**
```
cancer-care-miniapp/
├── src/
│   ├── pages/
│   │   ├── index.vue                      # 首页
│   │   ├── roadmap/
│   │   │   └── index.vue                  # Roadmap 时间轴
│   │   ├── articles/
│   │   │   ├── list.vue                   # 文章列表
│   │   │   ├── detail.vue                 # 文章详情 + 分享
│   │   │   └── search.vue                 # 搜索
│   │   ├── game/
│   │   │   ├── levels.vue                 # 关卡列表
│   │   │   ├── level-detail.vue           # 关卡游戏
│   │   │   └── achievement.vue            # 成就分享
│   │   ├── chat/
│   │   │   ├── index.vue                  # AI 助手
│   │   │   └── history.vue                # 对话历史 + 分享
│   │   ├── community/
│   │   │   ├── list.vue                   # 帖子列表
│   │   │   ├── post-detail.vue            # 帖子详情 + 分享
│   │   │   └── post-create.vue            # 发布帖子
│   │   ├── handbook/
│   │   │   └── index.vue                  # 新手手册
│   │   ├── auth/
│   │   │   ├── login.vue                  # 微信登录
│   │   │   └── identity-select.vue        # 身份选择
│   │   └── user/
│   │       ├── profile.vue                # 用户中心
│   │       ├── progress.vue               # 学习进度
│   │       ├── badges.vue                 # 勋章墙
│   │       └── saved.vue                  # 收藏列表
│   ├── components/
│   │   ├── ArticleCard.vue                # 文章卡片
│   │   ├── LevelCard.vue                  # 关卡卡片
│   │   ├── ChatMessage.vue                # 聊天气泡
│   │   ├── CommunityPost.vue              # 社区帖子卡片
│   │   ├── ShareModal.vue                 # 分享弹窗
│   │   ├── DisclaimerModal.vue            # 免责声明
│   │   └── SourceBadge.vue                # 信源等级徽章
│   ├── stores/
│   │   ├── auth.ts                        # 认证状态
│   │   ├── user.ts                        # 用户信息
│   │   ├── content.ts                     # 内容缓存
│   │   └── ai.ts                          # AI 配额状态
│   ├── services/
│   │   ├── api.ts                         # API 基础配置
│   │   ├── auth.ts                        # 认证服务
│   │   ├── content.ts                     # 内容服务
│   │   ├── game.ts                        # 游戏服务
│   │   ├── chat.ts                        # AI 服务
│   │   ├── community.ts                   # 社区服务
│   │   └── share.ts                       # 分享服务
│   ├── utils/
│   │   ├── http.ts                        # HTTP 拦截器
│   │   ├── storage.ts                     # 本地存储工具
│   │   ├── validators.ts                  # 验证函数
│   │   ├── analytics.ts                   # 分析埋点（仅匿名统计）
│   │   └── share-formatter.ts             # 分享文案格式化
│   ├── styles/
│   │   ├── variables.scss                 # CSS 变量
│   │   ├── global.scss                    # 全局样式
│   │   └── accessibility.scss             # 无障碍样式
│   └── app.vue                            # 应用入口
├── package.json
├── uni.config.js
├── tsconfig.json
└── .env.example
```

### 10.2 后端技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **框架** | Express/Koa | HTTP 服务器 |
| **数据库** | Supabase (PostgreSQL) | BaaS 平台 |
| **认证** | JWT | 无状态身份验证 |
| **文件存储** | Supabase Storage | 云存储 |
| **API 调用** | axios | HTTP 客户端 |
| **密码加密** | bcryptjs | 密码哈希 |
| **日志** | winston | 日志库 |
| **速率限制** | express-rate-limit | API 限流 |
| **环境变量** | dotenv | 配置管理 |
| **验证** | joi/yup | 数据验证 |
| **语言** | TypeScript | 类型安全 |

**后端目录结构：**
```
cancer-care-backend/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.ts             # 认证处理
│   │   ├── content.controller.ts          # 内容处理
│   │   ├── game.controller.ts             # 游戏处理
│   │   ├── chat.controller.ts             # AI 对话处理
│   │   ├── community.controller.ts        # 社区处理
│   │   ├── user.controller.ts             # 用户处理
│   │   ├── admin.controller.ts            # 管理后台处理
│   │   └── share.controller.ts            # 分享处理
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── content.service.ts
│   │   ├── game.service.ts
│   │   ├── chat.service.ts                # AI 集成（FastGPT/Dify）
│   │   ├── community.service.ts           # 先审后发逻辑
│   │   ├── user.service.ts
│   │   ├── admin.service.ts
│   │   ├── ai.service.ts                  # AI 调用 + 拒答过滤
│   │   ├── share.service.ts               # 分享链接生成
│   │   └── analytics.service.ts           # 数据分析
│   ├── models/
│   │   ├── user.model.ts
│   │   ├── article.model.ts
│   │   ├── game.model.ts
│   │   ├── chat.model.ts
│   │   └── community.model.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts             # JWT 验证
│   │   ├── error.middleware.ts            # 错误处理
│   │   ├── cors.middleware.ts             # CORS
│   │   ├── logger.middleware.ts           # 请求日志
│   │   ├── validation.middleware.ts       # 数据验证
│   │   └── rateLimit.middleware.ts        # 速率限制
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── content.routes.ts
│   │   ├── game.routes.ts
│   │   ├── chat.routes.ts
│   │   ├── community.routes.ts
│   │   ├── user.routes.ts
│   │   ├── admin.routes.ts
│   │   └── share.routes.ts
│   ├── utils/
│   │   ├── jwt.ts                         # JWT 工具
│   │   ├── wechat.ts                      # 微信 API
│   │   ├── supabase.ts                    # Supabase 客户端
│   │   ├── logger.ts                      # 日志工具
│   │   ├── validators.ts                  # 验证规则
│   │   └── sharing.ts                     # 分享链接工具
│   ├── config/
│   │   ├── database.ts                    # 数据库配置
│   │   ├── env.ts                         # 环境变量
│   │   └── constants.ts                   # 常量
│   ├── types/
│   │   └── index.ts                       # TypeScript 类型定义
│   └── app.ts                             # Express 应用
├── tests/
│   ├── unit/
│   ├── integration/
│   └── fixtures/
├── package.json
├── tsconfig.json
├── .env.example
├── Dockerfile
├── docker-compose.yml
└── README.md
```

---

## 11. 部署与运维

### 11.1 环境配置 (`.env.example`)

```bash
# 环境
NODE_ENV=production
PORT=3000

# Supabase
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx

# JWT
JWT_SECRET=your-secret-key
JWT_REFRESH_SECRET=your-refresh-secret

# 微信
WECHAT_APPID=xxx
WECHAT_APPSECRET=xxx
WECHAT_MP_TOKEN=xxx

# FastGPT/Dify
FASTGPT_API_KEY=xxx
FASTGPT_BASE_URL=https://xxx
DIFY_API_KEY=xxx
DIFY_BASE_URL=https://xxx

# 加密
ENCRYPT_KEY=your-encryption-key

# CORS
ALLOWED_ORIGINS=https://example.com,https://app.example.com

# 日志
LOG_LEVEL=info

# Analytics（匿名统计）
ENABLE_ANALYTICS=true
```

### 11.2 Docker 部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist
COPY .env ./.env

EXPOSE 3000

HEALTHCHECK --interval=30s --timeout=3s --start-period=40s --retries=3 \
  CMD node -e "require('http').get('http://localhost:3000/health', (r) => {if (r.statusCode !== 200) throw new Error(r.statusCode)})"

CMD ["node", "dist/app.js"]
```

```yaml
# docker-compose.yml
version: '3.8'
services:
  backend:
    build: .
    ports:
      - "3000:3000"
    environment:
      - NODE_ENV=production
    env_file:
      - .env
    restart: unless-stopped
    networks:
      - cancer-care
    healthcheck:
      test: ["CMD", "curl", "-f", "http://localhost:3000/health"]
      interval: 30s
      timeout: 3s
      retries: 3

networks:
  cancer-care:
```

### 11.3 CI/CD 流程 (GitHub Actions)

```yaml
# .github/workflows/deploy.yml
name: Deploy to Production

on:
  push:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '18'
      - run: npm ci
      - run: npm run test
      - run: npm run build

  deploy:
    needs: test
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      
      - name: Build Docker image
        run: docker build -t app:${{ github.sha }} .
      
      - name: Push to registry
        run: |
          docker tag app:${{ github.sha }} registry.example.com/app:latest
          docker push registry.example.com/app:latest
      
      - name: Deploy to production
        run: |
          ssh -i ${{ secrets.DEPLOY_KEY }} user@server \
            "cd /app && docker-compose pull && docker-compose up -d"
```

---

## 12. 版本节奏（公益版）

### Phase 1（MVP，4 周）
**目标**：建立最小可行产品，支持核心用户旅程

**前端交付**：
- [ ] 微信授权登录 + 身份选择
- [ ] Roadmap 时间轴（可视化展示）
- [ ] 文章列表 + 详情 + 分享功能
- [ ] AI 助手（10 次/日）+ 分享回复
- [ ] 工具跳转（公益外链）
- [ ] 用户中心（基础信息）

**后端交付**：
- [ ] Supabase 表初始化
- [ ] 用户认证 API
- [ ] 内容 API（文章、关卡、FAQ）
- [ ] AI 集成（FastGPT 或 Dify）+ 拒答日志
- [ ] 分享链接生成
- [ ] 基础数据分析

**运维交付**：
- [ ] GitHub 仓库 + CI/CD 配置
- [ ] Docker 镜像 + 部署脚本
- [ ] API 文档（Swagger）
- [ ] 小程序发布

---

### Phase 2（+6 周）
**目标**：完善社交与游戏化，强化内容审核

**前端交付**：
- [ ] 游戏化关卡完整体验
- [ ] 社区发帖 + 举报 + 分享
- [ ] 新手手册（H5 离线）
- [ ] 勋章系统 + 展示
- [ ] 数据看板 UI（用户视图）

**后端交付**：
- [ ] 社区先审后发工作流
- [ ] 内容审核流程（A/B/C 级双签）
- [ ] AI 拒答日志 + 人工复核
- [ ] 完整数据分析 API
- [ ] 用户反馈机制

**运维交付**：
- [ ] 监控告警配置
- [ ] 性能优化 + CDN 配置
- [ ] 备份策略
- [ ] 运维手册

---

### Phase 3（+4 周）
**目标**：无障碍、多语言、生态扩展

**前端交付**：
- [ ] 无障碍适配（屏幕阅读、字体放大）
- [ ] 多语言支持（中英文）
- [ ] 小程序二维码优化
- [ ] 离线内容缓存

**后端交付**：
- [ ] 多语言 API 支持
- [ ] 性能监控 + 告警
- [ ] 数据导出功能

**运维交付**：
- [ ] 生产就绪检查清单
- [ ] 灾难恢复 SOP
- [ ] 扩容方案

---

## 13. 关键成功指标（北极星）

| 指标 | 目标 | 监控周期 |
|------|------|--------|
| 新手任务完成率 | ≥ 55% | 每周 |
| 文章阅读完成率 | ≥ 60% | 每周 |
| 游戏关卡通过率 | ≥ 70% | 每周 |
| AI 拒答率 | ≤ 2% | 每日 |
| 社区举报率 | ≤ 1% | 每日 |
| 30 日留存 | ≥ 25% | 每周 |
| 文章分享率 | ≥ 15% | 每周 |
| AI 分享率 | ≥ 10% | 每周 |

---

## 14. 项目交付清单

### 前端（uni-app）
- [ ] 环境搭建 & 依赖配置
- [ ] 页面框架搭建（8 个主页面）
- [ ] API 集成 & 错误处理
- [ ] 状态管理 (Pinia) - auth、user、content、ai
- [ ] 样式和自定义 UI 组件
- [ ] 微信授权 & 登录集成
- [ ] 分享功能（文章、AI 回复、帖子、成就）
- [ ] 无障碍支持（ARIA 标签）
- [ ] 本地存储 & 缓存策略
- [ ] 性能优化（图片懒加载、代码分割）
- [ ] 本地测试 & Bug 修复
- [ ] 小程序发布

### 后端（Node.js）
- [ ] Express 服务器搭建
- [ ] Supabase 集成 & 表结构初始化
- [ ] 用户认证 API（微信、身份选择）
- [ ] 内容 CRUD API（文章、游戏、FAQ）
- [ ] 游戏逻辑 API（关卡答题、冷却机制）
- [ ] AI 集成 API（FastGPT/Dify 调用、拒答过滤）
- [ ] 社区 API（发帖、审核、举报、分享）
- [ ] 分享链接生成 & 追踪
- [ ] 用户信息 & 进度 API
- [ ] 数据导出 & 删除 API
- [ ] 管理后台 API（审核、数据看板、日志）
- [ ] 错误处理 & 日志系统
- [ ] 单元测试 & 集成测试
- [ ] Docker 配置
- [ ] 部署脚本 & 运维文档

### DevOps & 基础设施
- [ ] GitHub 仓库搭建 + branch 保护规则
- [ ] CI/CD 流程 (GitHub Actions)
- [ ] Docker 镜像构建 & 优化
- [ ] 云服务配置（Supabase、Vercel/Railway/ECS）
- [ ] 监控告警配置（Error tracking、API 延迟、资源使用）
- [ ] 日志聚合系统
- [ ] 备份策略 & 灾难恢复
- [ ] CDN 配置（静态资源加速）
- [ ] 性能基准测试

### 文档 & 合规
- [ ] API 文档（Swagger/OpenAPI）
- [ ] 数据库 ERD 图
- [ ] 架构设计文档
- [ ] 部署指南 & 运维手册
- [ ] 用户隐私政策 & 服务条款
- [ ] 医学顾问委员会章程
- [ ] AI 拒答模板库（加密存放）
- [ ] 内容审核指南
- [ ] 开发者文档 & 快速开始指南

### 测试 & 质保
- [ ] 功能测试（覆盖所有模块）
- [ ] 集成测试（API、数据库、第三方服务）
- [ ] 性能测试（负载、并发、响应时间）
- [ ] 安全测试（SQL 注入、XSS、CSRF、数据加密）
- [ ] 无障碍测试（屏幕阅读器、键盘导航）
- [ ] 兼容性测试（不同微信版本、手机型号）
- [ ] UAT（用户验收测试）

---

## 15. 成本预算

### 月度成本预算

| 项目 | 成本 (月) | 说明 |
|------|----------|------|
| **Supabase** | ¥0-500 | 免费计划 → Pro（根据数据量、存储、API 调用） |
| **服务器部署** | ¥0-300 | Vercel/Railway 免费或自建 ECS(¥50-200) |
| **CDN** | ¥50-200 | 小程序包体 + 静态资源加速 |
| **微信服务** | ¥0 | 官方免费，但小程序审核可能需审核员成本 |
| **FastGPT/Dify API** | ¥200-1000+ | 按提问量付费（10 次/日/用户，假设 10k 活跃用户 = 100k 提问/日） |
| **域名 & SSL** | ¥50 | 一次性购买，Let's Encrypt 免费续期 |
| **监控 & 日志** | ¥0-200 | 自建方案(¥0) 或 DataDog/New Relic(¥200+) |
| **医学顾问委员会** | ¥0 | 公益志愿者（或合作医院） |
| **运营人员** | 另计 | 内容编辑、审核员（如有人工成本） |
| **合计** | **¥300-2200+/月** | 初期低成本，可按需扩展 |

**成本优化建议**：
1. 前 3 个月使用免费套餐（Supabase 免费、Vercel 免费）
2. 使用开源监控（Prometheus + Grafana）
3. 医学顾问委员会由合作医院提供（公益）
4. AI API 采用按量付费模式，设置日配额上限

---

## 16. 配额管理补充说明

### 配额永久性承诺
- ✅ **每日免费 10 次**：每天 0 点重置，仅限当天使用
- ✅ **通过任务获得的配额**：永久有效，**绝不过期**
  - 完成关卡赚取 → 永久保留
  - 分享文章赚取 → 永久保留
  - 可累积无限多
- ⚠️ **用户不会因为"没用完"而失去已赚取的配额**

### 配额使用逻辑
```
用户发起 AI 提问
  ↓
检查今日免费配额（10 次）
  ├─ 还有免费次数 → 扣除 1 次免费
  └─ 免费用完 → 扣除累积奖励 1 次
  
用户会看到：
「今日免费：0/10 (已用完) | 累积奖励：46 次」
```

### 实际场景示例

**Day 1：**
```
基础配额：10 次
完成 1 个 easy 关卡：+10
分享 1 篇文章：+10
─────────────────
用完 15 次 AI 提问后：
  → 今日免费：0/10（清零）
  → 累积奖励：5 次（20-15=5）
  → 总剩余：5 次
```

**Day 2（第二天）：**
```
基础配额自动重置：10 次（✨ 新的免费额度）
累积奖励保留：5 次（✨ 昨天没用完的，今天继续用）
─────────────────
用户看到：
  「今日免费：10/10 | 累积奖励：5 次（昨日结转）| 总计：15 次」
  
用完 12 次后：
  → 今日免费：0/10（用完）
  → 累积奖励：-7 次（不够，扣除累积的 5 次）
  → 提示：「您的基础配额已用完，已为您扣除 5 次累积奖励，还剩 2 次需今日完成任务获得」
```

### 用户看到的配额显示（真实示例）

```
┌─────────────────────────────────────────┐
│  📊 我的 AI 配额                          │
├─────────────────────────────────────────┤
│                                          │
│  🎁 今日免费           ████░░░░░░  6/10 │
│     (今晚 24:00 清零)                    │
│                                          │
│  🏆 累积奖励            ███████████ 47   │
│     (永久有效，永不过期 ✨)              │
│                                          │
├─────────────────────────────────────────┤
│  💰 总计可用：53 次                      │
├─────────────────────────────────────────┤
│                                          │
│  如何获得更多奖励配额？                   │
│  ✓ 完成轻松关卡 → +10 (今日还可 2 次)   │
│  ✓ 完成挑战关卡 → +20 (今日还可 2 次)   │
│  ✓ 分享文章      → +10 (今日还可 2 次)   │
│                                          │
└─────────────────────────────────────────┘
```

---

## 17. 风险与应对

| 风险 | 影响 | 概率 | 应对方案 |
|------|------|------|--------|
| **AI 拒答率过高** | 用户体验差 | 中 | 定期调整 prompt、监控拒答日志、人工复核改进 |
| **监管政策变化** | 内容下线、业务中断 | 高 | 与医学顾问委员会月度评审、动态调整内容 |
| **医学顾问委员会变更** | 内容审核延迟 | 低 | 建立备选审核员名单、提前沟通 |
| **API 服务中断** | 用户无法使用功能 | 低 | 实现 Fallback 机制、多 API 备选、降级方案 |
| **数据泄露** | 隐私侵犯、法律风险 | 低 | 加密存储、访问控制、定期安全审计、保险 |
| **流量突增** | 系统崩溃 | 中 | 自动扩容、CDN 加速、速率限制、缓存策略 |
| **数据丢失** | 业务中断 | 低 | 每日自动备份、跨地域副本、定期恢复测试 |
| **第三方服务合作变更** | 工具链接失效 | 中 | 定期验证链接、建立备选合作方名单 |
| **用户采用率低** | 投入未见成效 | 中 | 早期 beta 测试、用户反馈循环、激励机制 |
| **医疗合规问题** | 下架、罚款 | 中 | 专业法律咨询、强化免责声明、内容严审 |

---

## 18. 项目时间线与里程碑

```
Week 1-2:  需求评审 + 技术选型 + 开发环境搭建
Week 3-4:  前后端基础框架 + Supabase 表设计
Week 5-6:  API 开发 + 微信集成 + 初版 UI
Week 7-8:  AI 集成 + 分享功能 + 错误处理
Week 9-10: 测试 + 性能优化 + 安全审计
Week 11-12: 上线前检查 + 小程序发布 + MVP 上线
```

**里程碑**：
- ✓ Phase 1 MVP 上线（第 4 周）
- ✓ Phase 2 完整版上线（第 10 周）
- ✓ Phase 3 扩展版上线（第 14 周）

---

## 19. 附录

### 附录 A：医学顾问委员会名单（待签字页）
```
主任：[医院名称] [主任名称]
成员：
- [三甲医院肿瘤科主任 1]
- [三甲医院肿瘤科主任 2]
- [患者代表 1]
- [患者代表 2]
```

### 附录 B：信源等级标签样式（UI 示意）
```
A 级: 红色标签 "权威指南"
B 级: 橙色标签 "专家署名"
C 级: 蓝色标签 "平台自产"
```

### 附录 C：AI 拒答 42 条模板（加密存放）
```
拒答模板分类：
1. 处方检测（10 条）
   - "不提供任何处方建议"
   - "药物剂量、用法需咨询医生"
   - ...
   
2. 诊断检测（8 条）
   - "无法进行线上诊断"
   - "症状判断需专业医评估"
   - ...
   
3. 手术决策（8 条）
   - "手术方案必须医生评估"
   - "个体化治疗方案无法给出"
   - ...
   
4. 医学伦理（16 条）
   - "无法替代医患沟通"
   - ...
```

### 附录 D：分享功能规范

**文章分享文案模板**：
```
我在【病情主动管理科普助手】看到一篇关于 [癌症类型] 的权威文章，
特别有帮助。推荐给你：[文章标题]

来自：[信源等级] - [出版来源]
```

**AI 回复分享文案模板**：
```
我在 AI 科普助手得到的回答，与你分享：

Q: [用户问题]
A: [AI 回复摘要，前 200 字]

⚠️ 本内容仅供参考，不能替代医生面诊。
```

**社区帖子分享文案模板**：
```
社区里的一则讨论，值得一看：[帖子摘要，前 150 字]

点击查看完整讨论 →
```

**成就分享文案模板**：
```
我在【病情主动管理科普助手】完成了知识闯关关卡，获得勋章！
你也来挑战吧 →

[勋章图标] [勋章名称]: [解锁条件]
```

---

## 20. 相关资源与参考

- **FastGPT**: https://fastgpt.run
- **Dify**: https://dify.ai
- **Supabase**: https://supabase.com
- **uni-app**: https://uniapp.dcloud.net.cn
- **WeChat 小程序文档**: https://developers.weixin.qq.com/miniprogram/dev
- **GDPR 合规指南**: https://gdpr-info.eu/
- **Node.js 最佳实践**: https://nodejs.org/en/docs/guides/
- **PostgreSQL 文档**: https://www.postgresql.org/docs/

---

**文档版本**: v2.0  
**最后更新**: 2025-12-23  
**维护者**: [项目团队]  
**下一次审核**: 2025-01-15
