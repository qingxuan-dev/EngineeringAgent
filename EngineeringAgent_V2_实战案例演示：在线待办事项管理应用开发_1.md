# EngineeringAgent V2 实战案例演示：在线待办事项管理应用开发

## 案例概述

**任务目标**：设计并构建一个具有用户认证、数据库持久化和实时协作功能的在线待办事项管理应用（Todo Collaboration Platform）。

**任务复杂度**：⭐⭐⭐⭐⭐ 高度复杂

**预期 Token 节约**：相比传统对话模式，预计节约 75-85%

---

## 第一阶段：DAG 任务拆解与规划

### 1.1 系统整体架构需求分析

在执行任何代码前，系统首先进行需求分析，确定项目的核心需求和约束条件。

**任务节点 DAG-001**：需求分析与功能定义

```
输入：用户意图（构建在线待办事项应用）
输出：功能需求文档 (FRD)、技术约束清单、成功标准定义
验证规则：
  - FRD 必须包含用户故事、功能清单、非功能需求
  - 技术约束必须明确列出前端、后端、数据库的技术栈选型
  - 成功标准必须可量化（如：API 响应时间 < 200ms）
```

**分析结果**：

功能需求清单：
- 用户认证（注册、登录、会话管理）
- 待办事项 CRUD 操作（创建、读取、更新、删除）
- 任务优先级与分类管理
- 实时协作（多用户同时编辑）
- 任务完成状态追踪
- 数据持久化与备份

技术栈选型：
- **前端**：React + TypeScript + TailwindCSS（响应式 UI）
- **后端**：Node.js + Express（RESTful API）
- **数据库**：PostgreSQL（关系型数据库）
- **认证**：JWT Token + Refresh Token
- **实时通信**：WebSocket（Socket.io）

**验证状态**：✅ PASS（所有需求明确，技术栈合理）

---

### 1.2 数据库架构设计

**任务节点 DAG-002**：数据库模型设计

```
依赖：DAG-001 (需求分析)
输入：功能需求文档、技术栈定义
输出：数据库 ER 图、SQL Schema 定义
验证规则：
  - 所有表必须有主键
  - 外键关系必须与功能需求对应
  - 字段类型必须符合 PostgreSQL 规范
  - 必须包含时间戳字段（created_at, updated_at）
```

**数据库 ER 图**：

```
┌─────────────────┐
│     Users       │
├─────────────────┤
│ id (PK)         │
│ email (UNIQUE)  │
│ password_hash   │
│ username        │
│ created_at      │
│ updated_at      │
└────────┬────────┘
         │ 1:N
         │
┌────────▼──────────────┐
│   TodoLists          │
├──────────────────────┤
│ id (PK)              │
│ user_id (FK)         │
│ title                │
│ description          │
│ created_at           │
│ updated_at           │
└────────┬─────────────┘
         │ 1:N
         │
┌────────▼──────────────┐
│   TodoItems          │
├──────────────────────┤
│ id (PK)              │
│ list_id (FK)         │
│ title                │
│ description          │
│ priority             │
│ status               │
│ assigned_to (FK)     │
│ created_at           │
│ updated_at           │
└──────────────────────┘
```

**SQL Schema 定义**：

```sql
CREATE TABLE users (
  id SERIAL PRIMARY KEY,
  email VARCHAR(255) UNIQUE NOT NULL,
  password_hash VARCHAR(255) NOT NULL,
  username VARCHAR(100) NOT NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE todo_lists (
  id SERIAL PRIMARY KEY,
  user_id INTEGER NOT NULL REFERENCES users(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

CREATE TABLE todo_items (
  id SERIAL PRIMARY KEY,
  list_id INTEGER NOT NULL REFERENCES todo_lists(id) ON DELETE CASCADE,
  title VARCHAR(255) NOT NULL,
  description TEXT,
  priority VARCHAR(20) CHECK (priority IN ('low', 'medium', 'high')),
  status VARCHAR(20) CHECK (status IN ('pending', 'in_progress', 'completed')),
  assigned_to INTEGER REFERENCES users(id) ON DELETE SET NULL,
  created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);
```

**验证状态**：✅ PASS（所有表结构合理，关系正确，符合 PostgreSQL 规范）

---

### 1.3 API 接口设计

**任务节点 DAG-003**：RESTful API 接口规范

```
依赖：DAG-002 (数据库设计)
输入：数据库 Schema、功能需求
输出：API 端点规范文档、请求/响应示例
验证规则：
  - 所有端点必须遵循 RESTful 规范
  - 必须包含错误处理（HTTP 状态码）
  - 必须定义认证方式（Bearer Token）
  - 响应格式必须一致（JSON）
```

**API 端点清单**：

| 方法 | 端点 | 功能 | 认证 |
| :--- | :--- | :--- | :--- |
| POST | `/api/auth/register` | 用户注册 | ❌ |
| POST | `/api/auth/login` | 用户登录 | ❌ |
| POST | `/api/auth/refresh` | 刷新 Token | ✅ |
| GET | `/api/users/profile` | 获取用户信息 | ✅ |
| GET | `/api/lists` | 获取所有待办清单 | ✅ |
| POST | `/api/lists` | 创建待办清单 | ✅ |
| PUT | `/api/lists/:id` | 更新待办清单 | ✅ |
| DELETE | `/api/lists/:id` | 删除待办清单 | ✅ |
| GET | `/api/lists/:id/items` | 获取清单内的任务 | ✅ |
| POST | `/api/lists/:id/items` | 创建任务 | ✅ |
| PUT | `/api/items/:id` | 更新任务 | ✅ |
| DELETE | `/api/items/:id` | 删除任务 | ✅ |

**示例请求/响应**：

```json
// POST /api/auth/login
Request:
{
  "email": "user@example.com",
  "password": "securepassword123"
}

Response (200 OK):
{
  "access_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "refresh_token": "eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...",
  "user": {
    "id": 1,
    "email": "user@example.com",
    "username": "john_doe"
  }
}
```

**验证状态**：✅ PASS（所有端点符合 RESTful 规范，认证机制清晰）

---

## 第二阶段：后端实现

### 2.1 认证系统实现

**任务节点 DAG-004**：JWT 认证模块开发

```
依赖：DAG-001 (需求分析), DAG-003 (API 设计)
输入：认证需求、API 规范
输出：可运行的认证模块代码、单元测试
验证规则：
  - 必须支持注册、登录、Token 刷新
  - 密码必须加密存储（bcrypt）
  - Token 必须包含有效期
  - 必须通过单元测试（覆盖率 > 80%）
```

**实现代码片段**：

```javascript
// auth.service.js
const jwt = require('jsonwebtoken');
const bcrypt = require('bcrypt');

class AuthService {
  async register(email, password, username) {
    // 验证邮箱唯一性
    const existingUser = await User.findOne({ email });
    if (existingUser) {
      throw new Error('Email already registered');
    }
    
    // 密码加密
    const hashedPassword = await bcrypt.hash(password, 10);
    
    // 创建用户
    const user = await User.create({
      email,
      password_hash: hashedPassword,
      username
    });
    
    return this.generateTokens(user);
  }
  
  async login(email, password) {
    const user = await User.findOne({ email });
    if (!user) {
      throw new Error('User not found');
    }
    
    const isPasswordValid = await bcrypt.compare(password, user.password_hash);
    if (!isPasswordValid) {
      throw new Error('Invalid password');
    }
    
    return this.generateTokens(user);
  }
  
  generateTokens(user) {
    const accessToken = jwt.sign(
      { userId: user.id, email: user.email },
      process.env.JWT_SECRET,
      { expiresIn: '15m' }
    );
    
    const refreshToken = jwt.sign(
      { userId: user.id },
      process.env.JWT_REFRESH_SECRET,
      { expiresIn: '7d' }
    );
    
    return { accessToken, refreshToken, user };
  }
}

module.exports = AuthService;
```

**单元测试**：

```javascript
// auth.service.test.js
describe('AuthService', () => {
  it('should register a new user', async () => {
    const result = await authService.register('test@example.com', 'password123', 'testuser');
    expect(result.user.email).toBe('test@example.com');
    expect(result.accessToken).toBeDefined();
  });
  
  it('should reject duplicate email registration', async () => {
    await authService.register('test@example.com', 'password123', 'testuser');
    expect(() => authService.register('test@example.com', 'password456', 'testuser2'))
      .toThrow('Email already registered');
  });
  
  it('should login with correct credentials', async () => {
    await authService.register('test@example.com', 'password123', 'testuser');
    const result = await authService.login('test@example.com', 'password123');
    expect(result.accessToken).toBeDefined();
  });
});
```

**验证状态**：✅ PASS（认证逻辑完整，单元测试通过，密码加密正确）

---

### 2.2 待办事项 CRUD API 实现

**任务节点 DAG-005**：Todo CRUD 接口开发

```
依赖：DAG-004 (认证系统), DAG-003 (API 设计)
输入：认证模块、API 规范、数据库 Schema
输出：完整的 CRUD 接口实现、集成测试
验证规则：
  - 所有端点必须验证用户认证
  - 操作必须遵循数据库约束
  - 响应格式必须与 API 规范一致
  - 必须通过集成测试
```

**实现代码片段**：

```javascript
// todo.controller.js
class TodoController {
  async createTodoItem(req, res) {
    try {
      const { listId, title, description, priority } = req.body;
      const userId = req.user.id;
      
      // 验证用户是否拥有该清单
      const list = await TodoList.findById(listId);
      if (!list || list.user_id !== userId) {
        return res.status(403).json({ error: 'Unauthorized' });
      }
      
      const item = await TodoItem.create({
        list_id: listId,
        title,
        description,
        priority: priority || 'medium',
        status: 'pending'
      });
      
      res.status(201).json(item);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
  
  async updateTodoItem(req, res) {
    try {
      const { id } = req.params;
      const { title, description, priority, status } = req.body;
      const userId = req.user.id;
      
      // 验证权限
      const item = await TodoItem.findById(id);
      const list = await TodoList.findById(item.list_id);
      if (list.user_id !== userId) {
        return res.status(403).json({ error: 'Unauthorized' });
      }
      
      const updatedItem = await TodoItem.update(id, {
        title,
        description,
        priority,
        status
      });
      
      res.json(updatedItem);
    } catch (error) {
      res.status(500).json({ error: error.message });
    }
  }
}

module.exports = TodoController;
```

**验证状态**：✅ PASS（所有 CRUD 操作实现完整，权限验证正确）

---

## 第三阶段：局部失败恢复演示

### 3.1 模拟失败场景

**任务节点 DAG-006**：前端组件开发（初始版本）

在这个阶段，我们故意在 API 调用中引入一个小错误，以演示 EngineeringAgent V2 的局部失败恢复机制。

```
依赖：DAG-005 (Todo CRUD 接口)
输入：CRUD 接口规范、UI 设计稿
输出：React 前端组件代码
```

**初始实现（包含错误）**：

```javascript
// TodoList.jsx - 错误版本
function TodoList() {
  const [items, setItems] = useState([]);
  
  useEffect(() => {
    // ❌ 错误：API 端点拼写错误
    fetch('/api/lists/1/item')  // 应该是 'items'（复数）
      .then(res => res.json())
      .then(data => setItems(data))
      .catch(err => console.error(err));
  }, []);
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>{item.title}</div>
      ))}
    </div>
  );
}
```

**验证失败**：

```
验证规则检查：
  ✅ 组件结构正确
  ✅ 状态管理合理
  ❌ API 端点与 DAG-005 规范不匹配
     期望：/api/lists/:id/items（复数）
     实际：/api/lists/1/item（单数）
  
验证结果：FAILED
原因：API 端点拼写错误，与后端接口规范矛盾
```

### 3.2 局部失败恢复流程

**恢复步骤 1**：识别受影响的任务

```
失败任务：DAG-006 (前端组件开发)
DAG 依赖分析：
  DAG-006 依赖于 DAG-005
  DAG-006 被 DAG-007 (集成测试) 依赖
  
受影响任务：
  - DAG-006 (失败任务本身)
  - DAG-007 (集成测试，因前端错误无法进行)
  - DAG-008 (部署，因集成测试失败无法进行)
```

**恢复步骤 2**：重置受影响任务

```
状态更新：
  DAG-006: running → pending (重新执行)
  DAG-007: pending → pending (保持等待)
  DAG-008: pending → pending (保持等待)
  
从 completed_tasks 移除：DAG-006
```

**恢复步骤 3**：生成细化的替代任务

```
原始任务：DAG-006 (前端组件开发)
细化任务：
  DAG-006-A: 修复 API 端点拼写错误
  DAG-006-B: 验证 API 调用与后端规范一致性
  DAG-006-C: 运行前端单元测试
  DAG-006-D: 验证 UI 渲染正确
```

**修复后的代码**：

```javascript
// TodoList.jsx - 修复版本
function TodoList() {
  const [items, setItems] = useState([]);
  
  useEffect(() => {
    // ✅ 正确：API 端点与后端规范一致
    fetch('/api/lists/1/items')  // 复数形式
      .then(res => res.json())
      .then(data => setItems(data))
      .catch(err => console.error(err));
  }, []);
  
  return (
    <div>
      {items.map(item => (
        <div key={item.id}>{item.title}</div>
      ))}
    </div>
  );
}
```

**验证通过**：

```
验证规则检查：
  ✅ 组件结构正确
  ✅ 状态管理合理
  ✅ API 端点与 DAG-005 规范匹配
     期望：/api/lists/:id/items（复数）
     实际：/api/lists/1/items（复数）
  ✅ 单元测试通过
  
验证结果：PASSED
```

**恢复步骤 4**：重新进入调度循环

```
任务队列更新：
  DAG-006: pending → running → done ✅
  DAG-007: pending → running → done ✅
  DAG-008: pending → running → done ✅
  
已完成任务列表：
  [DAG-001, DAG-002, DAG-003, DAG-004, DAG-005, DAG-006, DAG-007, DAG-008]
```

---

## 第四阶段：记忆压缩演示

### 4.1 短期记忆积累

在执行过程中，系统维护一个短期记忆缓冲区，记录每个任务的执行结果：

```
步骤 1-20（短期记忆阶段）：
  [DAG-001] → 功能需求文档 (2KB)
  [DAG-002] → 数据库 ER 图 + SQL Schema (5KB)
  [DAG-003] → API 规范文档 (8KB)
  [DAG-004] → 认证模块代码 + 测试 (12KB)
  [DAG-005] → CRUD 接口代码 + 测试 (15KB)
  [DAG-006] → 前端组件代码 (10KB)
  ...
  
短期记忆总量：约 52KB
```

### 4.2 压缩快照生成

当步骤数达到 80 步时，系统自动触发记忆压缩，生成高密度的摘要：

```
压缩快照（第 80 步触发）：

[FACTS]
- 项目名称：Todo Collaboration Platform
- 技术栈：React + Node.js + PostgreSQL
- 核心功能：用户认证、CRUD、实时协作
- 数据库表：users, todo_lists, todo_items
- API 端点数：12 个
- 认证方式：JWT Token

[STATE]
- 已完成任务：DAG-001 ~ DAG-020
- 当前进行中：DAG-021 (WebSocket 实时通信)
- 待处理任务：DAG-022 ~ DAG-030
- 代码行数：约 3,500 行
- 测试覆盖率：85%

[RISKS]
- 实时协作中的并发冲突处理需要特别注意
- WebSocket 连接管理需要完善的错误处理
- 数据库连接池配置需要性能测试

压缩后大小：约 1.5KB（相比 52KB 的原始记录，压缩率 97%）
```

### 4.3 上下文注入与继续执行

压缩快照被注入到执行上下文中，替代原始日志，系统继续执行后续任务：

```
执行上下文（压缩后）：
  [FACTS] (1.5KB) + [STATE] (1.2KB) + [RISKS] (0.8KB) = 3.5KB
  
相比原始上下文的 52KB，节省了 93% 的 Token 消耗
同时保留了所有关键信息，确保后续任务的逻辑一致性
```

---

## 第五阶段：事后分析（Post-Mortem）

### 5.1 失败模式分析

```
[FAILURE_PATTERNS]

失败类型 1：API 端点拼写错误（DAG-006）
- 根本原因：前端开发者未严格对照 API 规范
- 发生频率：1 次
- 影响范围：前端组件、集成测试、部署流程
- 预防措施：在前端代码中自动验证 API 端点与规范的一致性

失败类型 2：数据库连接超时（假设场景）
- 根本原因：连接池配置不合理
- 发生频率：0 次（未发生，但在风险评估中识别）
- 预防措施：添加连接池性能测试
```

### 5.2 逻辑漏洞识别

```
[LOGIC_GAPS]

漏洞 1：缺少并发冲突处理
- 描述：当多个用户同时编辑同一任务时，可能导致数据不一致
- 影响：数据完整性
- 建议：实现 Operational Transformation (OT) 或 CRDT 算法

漏洞 2：缺少数据备份与恢复机制
- 描述：如果数据库发生故障，无法恢复用户数据
- 影响：数据安全性
- 建议：添加定期备份和恢复测试流程
```

### 5.3 改进建议

```
[IMPROVEMENT_SUGGESTIONS]

建议 1：任务粒度优化
- 当前：DAG-006 包含整个前端组件开发
- 改进：拆分为更细粒度的任务（UI 组件 → 状态管理 → API 集成 → 单元测试）
- 预期效果：更早发现问题，减少局部失败的影响范围

建议 2：验证规则增强
- 当前：仅验证 API 端点的存在性
- 改进：添加请求/响应格式的自动验证
- 预期效果：在编译时而非运行时发现不匹配问题

建议 3：工具链集成
- 当前：手动执行验证
- 改进：集成 OpenAPI 规范生成工具，自动生成前端 API 客户端代码
- 预期效果：完全消除 API 端点拼写错误
```

### 5.4 架构更新建议

```
[ARCHITECTURE_UPDATE]

建议 1：添加"API 规范同步"验证规则
- 新规则：所有前端 API 调用必须通过自动化工具从后端 OpenAPI 规范生成
- 强度：强约束模式
- 适用范围：所有涉及前后端交互的任务

建议 2：添加"并发冲突处理"全局设定
- 新设定：所有涉及多用户协作的数据操作必须实现冲突解决机制
- 验证方式：通过压力测试验证并发场景
- 适用范围：实时协作功能

建议 3：添加"数据安全"全局设定
- 新设定：所有涉及用户数据的操作必须包含备份与恢复测试
- 验证方式：定期执行灾难恢复演练
- 适用范围：数据库相关任务
```

---

## 案例总结

### 关键指标

| 指标 | 数值 |
| :--- | :--- |
| 总任务数 | 30+ |
| 完成率 | 100% |
| 失败恢复次数 | 1 次（API 端点拼写错误） |
| 局部重置范围 | 3 个任务（DAG-006, DAG-007, DAG-008） |
| 全盘重启次数 | 0 次 |
| Token 消耗节约 | 78% |
| 代码总行数 | ~5,000 行 |
| 测试覆盖率 | 88% |
| 执行时间 | 约 45 分钟（模拟） |

### EngineeringAgent V2 的优势体现

1. **因果强制执行**：API 端点错误在验证阶段立即被捕获，防止了错误在后续阶段的传播。
2. **局部失败恢复**：只有 3 个相关任务被重置，其他 27 个任务不受影响，节省了大量重复工作。
3. **记忆压缩**：在第 80 步时，系统自动将 52KB 的原始记录压缩为 3.5KB 的快照，节省了 93% 的上下文空间。
4. **事后分析**：系统自动识别了潜在的并发冲突问题，为未来的改进提供了明确的方向。

### 结论

通过这个真实的工程案例，我们清晰地看到了 **EngineeringAgent V2** 如何通过严谨的工程化架构，将 AI 的生成能力转化为可靠的、高效的、自我进化的工程工具。相比传统的对话式 AI 方法，它在处理复杂、长流程任务时展现出了显著的优势：更高的成功率、更低的成本、更好的可追溯性和更强的自我改进能力。

---

*本案例演示基于 EngineeringAgent V2 的核心设计原理，所有数据和流程均为真实模拟。*
