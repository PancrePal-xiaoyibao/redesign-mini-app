# 癌症患者关护小程序设计文档

## 1. 产品概览

### 1.1 产品定位
一个面向癌症患者的综合关护平台，强调教育、社区支持、治疗资讯查询和第三方工具整合。**明确声明：此工具不提供医疗建议，仅为患者教育和信息聚合平台**。

### 1.2 核心价值
- **信息聚合**：集中呈现癌症相关的新闻、文章、知识
- **AI辅助**：通过FastGPT/Dify API提供问答支持（非医疗建议）
- **生态整合**：作为枢纽连接患者和现有功能性平台（AQ、好大夫、腾讯医典等）
- **隐私保护**：最小化个人敏感信息采集，降低GDPR风险
- **社区支持**：同伴支持、经验分享（后期迭代）

---

## 2. 核心功能模块

### 2.1 资讯中心 (News & Post)
**功能描述**
- 聚合癌症相关新闻、医学进展、患者故事
- 支持分类浏览（按癌症类型、主题、来源）
- 搜索功能
- 收藏和分享

**数据源**
- 编辑运营内容
- 第三方API集成（可选）
- 患者投稿（社区模式）

**权限**
- 游客可浏览（无需登录）
- 注册用户可收藏、评论（后期）

### 2.2 AI问答系统 (Q&A Chat)
**功能描述**
- 集成FastGPT或Dify API
- 患者输入问题→AI返回答案
- 会话历史记录（已登录用户）
- 常见问题(FAQ)库

**特点**
- **法律声明**：所有回答前需显示"此非医疗建议，请咨询医生"
- 支持文本输入，后期可支持语音
- 上下文记忆（个性化体验）
- 问题分类标签（方便追踪）

**API集成方式**
- FastGPT: 通过官方API调用
- Dify: 通过Workflow API调用
- 支持fallback机制（一个API故障时切换另一个）

### 2.3 应用中心 (App Hub)
**功能描述**
- 展示第三方工具和小程序的清单
- 快捷导航到：
  - 阿里巴巴 AQ（阿里互联网医疗平台）
  - 好大夫在线（医生咨询）
  - 腾讯医典（医疗科普）
  - 其他患者服务工具
- 后台可管理应用列表、排序、分类

**实现方式**
- 应用卡片展示（Logo、简介、跳转链接）
- 支持深度链接(Deep Link)或跳转URL
- 应用分类（咨询、用药、心理、生活、记录等）

### 2.4 用户中心 (User Center)
**功能描述**
- 个人资料管理（最小化敏感信息）
- 登出
- 偏好设置
- 隐私政策同意

**存储信息**
- 用户ID、用户名、邮箱、手机号（可选）
- 癌症类型（可选，用于个性化内容推荐）
- 注册方式（微信/邮箱/手机）
- 创建/更新时间

---

## 3. 技术架构

### 3.1 技术栈

| 层级 | 技术 | 说明 |
|------|------|------|
| **前端** | uni-app + Vue 3 | 跨平台小程序框架 |
| **前端状态管理** | Pinia | 状态管理库 |
| **后端框架** | Node.js + Express/Koa | RESTful API |
| **数据库** | Supabase (PostgreSQL) | BaaS平台 |
| **认证** | Supabase Auth + JWT | 用户认证 |
| **文件存储** | Supabase Storage | 用户上传、资源存储 |
| **第三方API** | FastGPT / Dify | AI问答 |
| **微信集成** | 微信开放平台SDK | 登录、分享等 |
| **部署** | Docker + Vercel/Railway/自建服务器 | 容器化部署 |

### 3.2 系统架构图

```
┌─────────────────────────────────────────────────────────┐
│                     WeChat Mini-App                      │
│                  (uni-app + Vue 3)                       │
│  ┌──────────────┬──────────────┬──────────────┐         │
│  │   资讯中心   │   AI问答     │   应用中心   │         │
│  │  用户中心    │              │              │         │
│  └──────────────┴──────────────┴──────────────┘         │
└─────────────┬───────────────────────────────────────────┘
              │ HTTPS REST API
              │
┌─────────────▼───────────────────────────────────────────┐
│           Node.js Backend (Express/Koa)                 │
│  ┌──────────────────────────────────────────────────┐  │
│  │  Routes: /auth /news /chat /apps /users          │  │
│  │  Middleware: Auth, Logging, CORS, Validation     │  │
│  │  Services: UserService, NewsService, etc         │  │
│  └──────────────────────────────────────────────────┘  │
└─────────────┬──────────────────────────────────────────┘
              │
    ┌─────────┼─────────┬──────────┐
    │         │         │          │
┌───▼──┐  ┌──▼───┐ ┌───▼──┐ ┌────▼──────┐
│ Supa │  │微信  │ │FastGPT│ │  Dify    │
│ base │  │API   │ │ API   │ │  API     │
└──────┘  └──────┘ └───────┘ └──────────┘
```

### 3.3 部署架构

```
GitHub → GitHub Actions (CI/CD)
         ↓
    ┌────────────┐
    │  Docker    │
    │ Image      │
    └────────────┘
         ↓
┌──────────────────────┐
│   Vercel/Railway     │
│   or Self-hosted     │
│   (Node.js Server)   │
└──────────────────────┘
         │
    ┌────┴────┐
    ↓         ↓
 Supabase   WeChat
 (Database) (OAuth)
```

---

## 4. 数据库设计 (Supabase PostgreSQL)

### 4.1 表结构

#### 表 1: `users`
```sql
CREATE TABLE users (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  username VARCHAR(255) NOT NULL UNIQUE,
  email VARCHAR(255),
  phone VARCHAR(20),
  wechat_id VARCHAR(255),
  cancer_type VARCHAR(100),  -- 可选：肺癌、乳腺癌、胰腺癌等
  registration_method VARCHAR(50),  -- 'wechat' | 'email' | 'phone'
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW(),
  deleted_at TIMESTAMP  -- 软删除
);

CREATE INDEX idx_users_email ON users(email);
CREATE INDEX idx_users_wechat_id ON users(wechat_id);
```

#### 表 2: `news`
```sql
CREATE TABLE news (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  title VARCHAR(500) NOT NULL,
  content TEXT NOT NULL,
  summary VARCHAR(1000),
  category VARCHAR(100),  -- '新闻'|'科普'|'进展'|'患者故事'
  cancer_types TEXT[],  -- ARRAY['肺癌', '乳腺癌']
  source VARCHAR(255),
  author_id UUID REFERENCES users(id) ON DELETE SET NULL,  -- 如果是用户投稿
  is_published BOOLEAN DEFAULT false,
  published_at TIMESTAMP,
  view_count INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_news_category ON news(category);
CREATE INDEX idx_news_published ON news(is_published, published_at DESC);
```

#### 表 3: `saved_articles`
```sql
CREATE TABLE saved_articles (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  news_id UUID NOT NULL REFERENCES news(id) ON DELETE CASCADE,
  created_at TIMESTAMP DEFAULT NOW(),
  UNIQUE(user_id, news_id)
);

CREATE INDEX idx_saved_articles_user ON saved_articles(user_id);
```

#### 表 4: `chat_conversations`
```sql
CREATE TABLE chat_conversations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255),  -- 会话标题，自动从第一条消息生成
  ai_provider VARCHAR(50),  -- 'fastgpt' | 'dify'
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_chat_user ON chat_conversations(user_id);
```

#### 表 5: `chat_messages`
```sql
CREATE TABLE chat_messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  conversation_id UUID NOT NULL REFERENCES chat_conversations(id) ON DELETE CASCADE,
  user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  role VARCHAR(50),  -- 'user' | 'assistant'
  content TEXT NOT NULL,
  message_type VARCHAR(50) DEFAULT 'text',  -- 'text' | 'image' | 'voice'
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_chat_messages_conversation ON chat_messages(conversation_id);
CREATE INDEX idx_chat_messages_user ON chat_messages(user_id);
```

#### 表 6: `apps`
```sql
CREATE TABLE apps (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name VARCHAR(255) NOT NULL,
  description TEXT,
  logo_url VARCHAR(500),
  category VARCHAR(100),  -- '咨询'|'用药'|'心理'|'生活'|'记录'等
  target_url VARCHAR(500) NOT NULL,  -- 跳转链接或Deep Link
  is_active BOOLEAN DEFAULT true,
  sort_order INT DEFAULT 0,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_apps_category ON apps(category);
CREATE INDEX idx_apps_sort ON apps(sort_order);
```

#### 表 7: `faqs`
```sql
CREATE TABLE faqs (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  question VARCHAR(500) NOT NULL,
  answer TEXT NOT NULL,
  category VARCHAR(100),
  sort_order INT DEFAULT 0,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT NOW(),
  updated_at TIMESTAMP DEFAULT NOW()
);

CREATE INDEX idx_faqs_category ON faqs(category);
```

---

## 5. API 设计

### 5.1 认证模块 `/api/auth`

#### 5.1.1 微信一键登录
```
POST /api/auth/wechat-login
Content-Type: application/json

{
  "code": "微信授权code",
  "encryptedData": "加密数据",
  "iv": "初始向量"
}

Response (200):
{
  "success": true,
  "data": {
    "access_token": "jwt_token",
    "refresh_token": "refresh_token",
    "user": {
      "id": "uuid",
      "username": "username",
      "wechat_id": "wx_123456"
    }
  }
}
```

#### 5.1.2 邮箱/手机注册
```
POST /api/auth/register
{
  "username": "user123",
  "email": "user@example.com",
  "phone": "13800138000",
  "password": "hashed_password"
}

Response (201):
{ 
  "success": true,
  "data": { "user_id": "uuid", "email": "user@example.com" }
}
```

#### 5.1.3 登录
```
POST /api/auth/login
{
  "email": "user@example.com",
  "password": "hashed_password"
}

Response (200):
{
  "success": true,
  "data": {
    "access_token": "jwt_token",
    "user": { ... }
  }
}
```

#### 5.1.4 登出
```
POST /api/auth/logout
Headers: { "Authorization": "Bearer token" }

Response (200):
{ "success": true }
```

---

### 5.2 资讯模块 `/api/news`

#### 5.2.1 获取资讯列表
```
GET /api/news?category=科普&cancer_type=肺癌&page=1&limit=10

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "肺癌新疗法...",
      "summary": "...",
      "category": "科普",
      "published_at": "2025-01-15T10:00:00Z",
      "view_count": 234
    }
  ],
  "pagination": {
    "page": 1,
    "limit": 10,
    "total": 100
  }
}
```

#### 5.2.2 获取资讯详情
```
GET /api/news/:id

Response (200):
{
  "success": true,
  "data": {
    "id": "uuid",
    "title": "...",
    "content": "...",
    "category": "...",
    "source": "...",
    "created_at": "...",
    "is_saved": false  -- 当前用户是否已收藏
  }
}
```

#### 5.2.3 搜索资讯
```
GET /api/news/search?q=关键词&page=1

Response (200):
{
  "success": true,
  "data": [ ... ]
}
```

#### 5.2.4 收藏资讯
```
POST /api/news/:id/save
Headers: { "Authorization": "Bearer token" }

Response (201):
{ "success": true }
```

#### 5.2.5 取消收藏
```
DELETE /api/news/:id/save
Headers: { "Authorization": "Bearer token" }

Response (200):
{ "success": true }
```

---

### 5.3 AI问答模块 `/api/chat`

#### 5.3.1 创建对话
```
POST /api/chat/conversations
Headers: { "Authorization": "Bearer token" }
{
  "title": "关于肺癌治疗的问题"
}

Response (201):
{
  "success": true,
  "data": {
    "conversation_id": "uuid",
    "title": "关于肺癌治疗的问题"
  }
}
```

#### 5.3.2 发送消息
```
POST /api/chat/conversations/:conversation_id/messages
Headers: { "Authorization": "Bearer token" }
{
  "content": "肺癌的早期症状有哪些？"
}

Response (200):
{
  "success": true,
  "data": {
    "user_message": {
      "id": "msg_1",
      "role": "user",
      "content": "肺癌的早期症状有哪些？",
      "created_at": "..."
    },
    "assistant_message": {
      "id": "msg_2",
      "role": "assistant",
      "content": "早期肺癌通常没有明显症状...（法律免责声明已包含）",
      "created_at": "..."
    }
  }
}
```

#### 5.3.3 获取对话历史
```
GET /api/chat/conversations/:conversation_id/messages?page=1&limit=20
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": [
    { "role": "user", "content": "...", "created_at": "..." },
    { "role": "assistant", "content": "...", "created_at": "..." }
  ]
}
```

#### 5.3.4 获取用户的所有对话
```
GET /api/chat/conversations
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "title": "关于肺癌治疗的问题",
      "updated_at": "...",
      "message_count": 5
    }
  ]
}
```

#### 5.3.5 删除对话
```
DELETE /api/chat/conversations/:conversation_id
Headers: { "Authorization": "Bearer token" }

Response (200):
{ "success": true }
```

---

### 5.4 应用中心模块 `/api/apps`

#### 5.4.1 获取应用列表
```
GET /api/apps?category=咨询&page=1&limit=20

Response (200):
{
  "success": true,
  "data": [
    {
      "id": "uuid",
      "name": "好大夫在线",
      "description": "在线医生咨询平台",
      "logo_url": "https://...",
      "category": "咨询",
      "target_url": "https://..."
    }
  ]
}
```

#### 5.4.2 获取应用分类
```
GET /api/apps/categories

Response (200):
{
  "success": true,
  "data": ["咨询", "用药", "心理", "生活", "记录"]
}
```

---

### 5.5 用户模块 `/api/users`

#### 5.5.1 获取个人信息
```
GET /api/users/profile
Headers: { "Authorization": "Bearer token" }

Response (200):
{
  "success": true,
  "data": {
    "id": "uuid",
    "username": "user123",
    "email": "user@example.com",
    "phone": "138...",
    "cancer_type": "肺癌",
    "created_at": "..."
  }
}
```

#### 5.5.2 更新个人信息
```
PUT /api/users/profile
Headers: { "Authorization": "Bearer token" }
{
  "username": "new_username",
  "cancer_type": "乳腺癌"
}

Response (200):
{ "success": true }
```

#### 5.5.3 修改密码
```
POST /api/users/change-password
Headers: { "Authorization": "Bearer token" }
{
  "old_password": "...",
  "new_password": "..."
}

Response (200):
{ "success": true }
```

---

### 5.6 管理员接口 `/api/admin`

#### 5.6.1 创建资讯
```
POST /api/admin/news
Headers: { "Authorization": "Bearer admin_token" }
{
  "title": "...",
  "content": "...",
  "category": "科普",
  "cancer_types": ["肺癌"],
  "source": "...",
  "is_published": true
}

Response (201):
{ "success": true, "data": { "news_id": "uuid" } }
```

#### 5.6.2 更新应用列表
```
PUT /api/admin/apps/:id
Headers: { "Authorization": "Bearer admin_token" }
{
  "name": "...",
  "sort_order": 1
}

Response (200):
{ "success": true }
```

#### 5.6.3 获取系统统计
```
GET /api/admin/stats
Headers: { "Authorization": "Bearer admin_token" }

Response (200):
{
  "success": true,
  "data": {
    "total_users": 5000,
    "active_users_today": 1200,
    "total_news": 450,
    "total_conversations": 8000
  }
}
```

---

## 6. 用户流程 (User Flows)

### 6.1 注册与登录流程

```
游客
  ↓
[点击"登录/注册"]
  ↓
┌─────────────────────────┐
│ 选择登录方式：          │
│ 1. 微信一键登录        │
│ 2. 邮箱注册/登录       │
│ 3. 手机号注册/登录     │
└─────────────────────────┘
  ↓
[微信授权] → [创建用户] → [生成JWT Token] → [跳转首页]
  ↓
已登录用户
```

### 6.2 资讯浏览流程

```
首页
  ↓
[分类筛选] [搜索] [推荐]
  ↓
资讯列表
  ↓
[点击资讯]
  ↓
资讯详情页
  ├─ [收藏] → 保存到"我的收藏"
  ├─ [分享] → 分享给微信好友/朋友圈
  └─ [返回]

我的收藏页面
  ↓
[展示已收藏的资讯]
  ├─ [删除]
  └─ [重新阅读]
```

### 6.3 AI问答流程

```
[进入AI问答]
  ↓
[需要登录？]
├─ 是 → [跳转登录]
└─ 否 → 继续
  ↓
[选择对话或创建新对话]
  ↓
[输入问题]
  ↓
[显示法律声明："此非医疗建议..."]
  ↓
[调用FastGPT/Dify API]
  ↓
[展示回答]
  ↓
[用户可继续提问 或 查看历史对话]
  ↓
[保存对话（自动）]
```

### 6.4 应用中心流程

```
[进入应用中心]
  ↓
[浏览应用列表] [按分类筛选]
  ↓
[查看应用卡片（Logo、名称、简介）]
  ↓
[点击"打开应用" 或 "了解更多"]
  ↓
[跳转到第三方应用]
  │（Deep Link 或 浏览器打开）
  ↓
[用户使用第三方应用]
  ↓
[返回小程序（可选）]
```

---

## 7. 前端架构 (uni-app)

### 7.1 目录结构

```
cancer-care-miniapp/
├── src/
│   ├── pages/
│   │   ├── index.vue                  # 首页
│   │   ├── news/
│   │   │   ├── list.vue              # 资讯列表
│   │   │   ├── detail.vue            # 资讯详情
│   │   │   └── search.vue            # 搜索
│   │   ├── chat/
│   │   │   ├── conversations.vue     # 对话列表
│   │   │   └── chat-detail.vue       # 对话详情
│   │   ├── apps/
│   │   │   └── list.vue              # 应用中心
│   │   ├── auth/
│   │   │   ├── login.vue             # 登录
│   │   │   ├── register.vue          # 注册
│   │   │   └── wechat-login.vue      # 微信登录
│   │   └── user/
│   │       └── profile.vue           # 用户中心
│   ├── components/
│   │   ├── NewsCard.vue              # 资讯卡片
│   │   ├── ChatMessage.vue           # 聊天消息
│   │   ├── AppCard.vue               # 应用卡片
│   │   └── DisclaimerModal.vue       # 法律声明弹窗
│   ├── stores/
│   │   ├── auth.ts                   # 认证状态 (Pinia)
│   │   ├── user.ts                   # 用户信息
│   │   └── chat.ts                   # 聊天状态
│   ├── services/
│   │   ├── api.ts                    # API 基础配置
│   │   ├── auth.ts                   # 认证服务
│   │   ├── news.ts                   # 资讯服务
│   │   ├── chat.ts                   # 聊天服务
│   │   └── apps.ts                   # 应用服务
│   ├── utils/
│   │   ├── http.ts                   # HTTP 客户端
│   │   ├── storage.ts                # 本地存储
│   │   ├── wechat.ts                 # 微信API
│   │   └── validators.ts             # 验证函数
│   ├── styles/
│   │   ├── variables.scss            # CSS 变量
│   │   └── global.scss               # 全局样式
│   └── app.vue                       # 应用入口
├── package.json
├── uni.config.js
├── tsconfig.json
└── .env.example
```

### 7.2 关键技术决策

| 方面 | 选择 | 理由 |
|------|------|------|
| 状态管理 | Pinia | 比Vuex更轻量，对Vue 3更友好 |
| 网络请求 | axios | 广泛应用，易于拦截和错误处理 |
| 本地存储 | uni-app Storage API | 跨平台支持 |
| 日期处理 | dayjs | 轻量级，无依赖 |
| UI框架 | uni-ui / 自定义 | 自定义以满足品牌要求 |

---

## 8. 后端架构 (Node.js)

### 8.1 目录结构

```
cancer-care-backend/
├── src/
│   ├── controllers/
│   │   ├── auth.controller.ts
│   │   ├── news.controller.ts
│   │   ├── chat.controller.ts
│   │   ├── apps.controller.ts
│   │   └── user.controller.ts
│   ├── services/
│   │   ├── auth.service.ts
│   │   ├── news.service.ts
│   │   ├── chat.service.ts
│   │   ├── app.service.ts
│   │   ├── user.service.ts
│   │   └── ai.service.ts            # FastGPT/Dify 集成
│   ├── models/
│   │   ├── user.model.ts
│   │   ├── news.model.ts
│   │   ├── chat.model.ts
│   │   └── app.model.ts
│   ├── middleware/
│   │   ├── auth.middleware.ts       # JWT 验证
│   │   ├── error.middleware.ts      # 错误处理
│   │   ├── cors.middleware.ts       # CORS
│   │   └── logger.middleware.ts     # 日志
│   ├── routes/
│   │   ├── auth.routes.ts
│   │   ├── news.routes.ts
│   │   ├── chat.routes.ts
│   │   ├── apps.routes.ts
│   │   ├── users.routes.ts
│   │   └── admin.routes.ts
│   ├── utils/
│   │   ├── jwt.ts                   # JWT工具
│   │   ├── wechat.ts                # 微信API
│   │   ├── supabase.ts              # Supabase客户端
│   │   └── logger.ts                # 日志
│   ├── config/
│   │   ├── database.ts              # 数据库配置
│   │   ├── env.ts                   # 环境变量
│   │   └── constants.ts             # 常量
│   ├── types/
│   │   └── index.ts                 # TypeScript类型定义
│   └── app.ts                       # Express 应用
├── package.json
├── tsconfig.json
├── .env.example
├── dockerfile
└── docker-compose.yml
```

### 8.2 技术栈详情

```javascript
// package.json (关键依赖)
{
  "dependencies": {
    "express": "^4.18.0",
    "@supabase/supabase-js": "^2.38.0",
    "jsonwebtoken": "^9.1.0",
    "axios": "^1.6.0",           // 调用第三方API
    "bcryptjs": "^2.4.3",         // 密码加密
    "dotenv": "^16.3.1",
    "cors": "^2.8.5",
    "uuid": "^9.0.0"
  },
  "devDependencies": {
    "typescript": "^5.0.0",
    "@types/express": "^4.17.0",
    "ts-node": "^10.9.0",
    "nodemon": "^3.0.0"
  }
}
```

### 8.3 FastGPT/Dify 集成方式

#### 调用 FastGPT
```typescript
// services/ai.service.ts
import axios from 'axios';

class AIService {
  private fastgptApiKey = process.env.FASTGPT_API_KEY;
  private fastgptBaseURL = process.env.FASTGPT_BASE_URL;
  
  async askFastGPT(question: string): Promise<string> {
    try {
      const response = await axios.post(
        `${this.fastgptBaseURL}/api/v1/chat`,
        {
          messages: [{ role: 'user', content: question }]
        },
        {
          headers: {
            'Authorization': `Bearer ${this.fastgptApiKey}`,
            'Content-Type': 'application/json'
          }
        }
      );
      return response.data.choices[0].message.content;
    } catch (error) {
      console.error('FastGPT API error:', error);
      return '抱歉，无法获取答案。请稍后重试。';
    }
  }
}
```

#### 调用 Dify
```typescript
async askDify(question: string, conversationId?: string): Promise<{
  answer: string,
  conversationId: string
}> {
  try {
    const response = await axios.post(
      `${process.env.DIFY_BASE_URL}/api/v1/chat-messages`,
      {
        inputs: {},
        query: question,
        response_mode: 'blocking',
        conversation_id: conversationId || undefined,
        user: 'user-id'
      },
      {
        headers: {
          'Authorization': `Bearer ${process.env.DIFY_API_KEY}`,
          'Content-Type': 'application/json'
        }
      }
    );
    
    return {
      answer: response.data.answer,
      conversationId: response.data.conversation_id
    };
  } catch (error) {
    // Fallback to FastGPT or return error message
    throw new Error('Dify API error');
  }
}
```

---

## 9. 数据安全与隐私

### 9.1 隐私设计原则

| 原则 | 实现 |
|------|------|
| **最小化数据采集** | 仅采集必要信息：用户名、邮箱/手机/微信ID |
| **不采集敏感医疗数据** | 不存储病历、用药记录、诊断结果 |
| **透明同意** | 注册时明确告知数据用途，获得明确同意 |
| **数据加密** | 传输层：HTTPS；存储层：敏感字段加密 |
| **访问控制** | 用户仅可访问自己的数据 |
| **数据出口** | 用户可导出或删除自己的数据 |

### 9.2 GDPR 合规性

- **个人数据处理协议**：与Supabase签署DPA
- **用户权利**：提供数据导出、删除、修正功能
- **隐私政策**：清晰说明数据处理方式
- **软删除机制**：用户可请求数据删除（30天延迟）
- **第三方API声明**：明确说明FastGPT/Dify可能接收的数据

### 9.3 安全措施

```typescript
// 1. 密码加密
import bcryptjs from 'bcryptjs';
const hashedPassword = await bcryptjs.hash(password, 10);

// 2. JWT Token (有效期短)
const token = jwt.sign(
  { userId: user.id },
  process.env.JWT_SECRET,
  { expiresIn: '1h' }  // 1小时过期，需要刷新
);

// 3. HTTPS 强制
// 通过环境变量和代理配置

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
```

### 9.4 法律声明

所有AI回答前需显示：
```
⚠️ 重要声明：
本工具基于AI技术提供的信息仅供参考，不构成医疗建议。
请勿将本工具的回答作为医疗诊断或治疗的依据。
如有任何健康疑虑，请立即咨询专业医疗人士。
```

---

## 10. 部署与运维

### 10.1 环境配置

```bash
# .env.example
NODE_ENV=production
PORT=3000

# Supabase
SUPABASE_URL=https://xxxx.supabase.co
SUPABASE_ANON_KEY=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx

# JWT
JWT_SECRET=your-secret-key
JWT_REFRESH_SECRET=your-refresh-secret

# WeChat
WECHAT_APPID=xxx
WECHAT_APPSECRET=xxx
WECHAT_MP_TOKEN=xxx

# FastGPT
FASTGPT_API_KEY=xxx
FASTGPT_BASE_URL=https://xxx

# Dify
DIFY_API_KEY=xxx
DIFY_BASE_URL=https://xxx

# CORS
ALLOWED_ORIGINS=https://example.com,https://app.example.com

# Log
LOG_LEVEL=info
```

### 10.2 Docker 部署

```dockerfile
# Dockerfile
FROM node:18-alpine

WORKDIR /app

COPY package*.json ./
RUN npm ci --only=production

COPY dist ./dist
COPY .env ./.env

EXPOSE 3000

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
```

### 10.3 CI/CD 流程

```yaml
# .github/workflows/deploy.yml
name: Deploy

on:
  push:
    branches: [main]

jobs:
  build-and-deploy:
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
          # 使用 SSH 连接到服务器并更新容器
          ssh -i ${{ secrets.DEPLOY_KEY }} user@server "cd /app && docker-compose pull && docker-compose up -d"
```

---

## 11. 开发路线图

### Phase 1: MVP (第1-2个月)
- [ ] 基础用户认证（微信、邮箱）
- [ ] 资讯模块（展示、分类、搜索）
- [ ] AI问答模块（FastGPT集成）
- [ ] 应用中心（展示常用工具）
- [ ] 用户中心（基础信息管理）
- [ ] 部署到生产环境

### Phase 2: 优化 & 扩展 (第3-4个月)
- [ ] Dify API 作为备选
- [ ] 收藏功能完整化
- [ ] 搜索优化（全文搜索）
- [ ] 分享功能（微信、QQ）
- [ ] 用户反馈机制
- [ ] 性能优化、缓存策略

### Phase 3: 社区与增值 (第5-6个月)
- [ ] 患者故事分享模块
- [ ] 评论和点赞功能
- [ ] 社区讨论板块
- [ ] 数据统计和insights
- [ ] 推荐算法

### Phase 4: 生态扩展 (后续)
- [ ] 更多第三方应用集成
- [ ] 微信支付集成（捐赠）
- [ ] 小程序分享卡片优化
- [ ] 用户增长运营

---

## 12. 项目清单

### 前端 (uni-app)
- [ ] 环境搭建 & 依赖配置
- [ ] 页面框架搭建
- [ ] API 集成
- [ ] 状态管理 (Pinia)
- [ ] 样式和UI组件
- [ ] 微信授权集成
- [ ] 本地测试
- [ ] 小程序发布

### 后端 (Node.js)
- [ ] Express 服务器搭建
- [ ] Supabase 集成
- [ ] 数据库表结构设计
- [ ] API 路由编写
- [ ] 认证中间件
- [ ] FastGPT/Dify 集成
- [ ] 错误处理和日志
- [ ] 单元测试和集成测试
- [ ] Docker 配置
- [ ] 部署到云平台

### DevOps
- [ ] GitHub 仓库设置
- [ ] CI/CD 流程
- [ ] 监控和告警
- [ ] 备份策略

### 文档
- [ ] API 文档（Swagger/OpenAPI）
- [ ] 用户隐私政策
- [ ] 用户使用指南
- [ ] 运维手册

---

## 13. 成本估算

| 项目 | 成本 (月) | 说明 |
|------|----------|------|
| Supabase | ¥0-500 | 免费计划到Pro（根据数据量） |
| 服务器部署 | ¥50-500 | Vercel/Railway免费或自建ECS |
| 域名 | ¥50-100 | 一次性，可选 |
| SSL证书 | ¥0 | Let's Encrypt 免费 |
| FastGPT/Dify | ¥0-1000+ | 按API调用量付费 |
| **总计** | **¥100-2000+** | 初期低成本，可扩展 |

---

## 14. 风险与应对

| 风险 | 影响 | 应对方案 |
|------|------|--------|
| **API 依赖风险** | FastGPT/Dify 宕机 | 实现Fallback机制、多API备选 |
| **监管风险** | 医疗广告/建议问题 | 强化法律声明、定期审查内容 |
| **隐私泄露** | 用户信息泄露 | 加密、访问控制、定期安全审计 |
| **流量突增** | 系统崩溃 | CDN加速、自动扩容、速率限制 |
| **数据丢失** | 业务中断 | 定期备份、灾备方案 |

---

## 15. 相关资源与参考

- **FastGPT**: https://fastgpt.run
- **Dify**: https://dify.ai
- **Supabase**: https://supabase.com
- **uni-app**: https://uniapp.dcloud.net.cn
- **WeChat 小程序文档**: https://developers.weixin.qq.com/miniprogram/dev
- **Node.js 最佳实践**: https://nodejs.org/en/docs/guides/nodejs-application-architecture
- **GDPR 合规**: https://gdpr-info.eu/

---

**文档版本**: v1.0  
**最后更新**: 2025-01-15  
**维护者**: [项目团队]
