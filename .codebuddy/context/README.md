# 上下文管理说明

## 目录结构

```
.codebuddy/context/
├── template.json           # 上下文模板
├── session_YYYY-MM-DD_01.json  # 对话上下文文件
├── session_YYYY-MM-DD_02.json  # 同一天的第二个对话
└── README.md              # 本说明文件
```

## 上下文文件命名规则

- 格式：`session_{日期}_{序号}.json`
- 日期格式：YYYY-MM-DD（例如：2025-01-19）
- 序号格式：两位数字，从01开始递增
- 示例：
  - `session_2025-01-19_01.json` - 2025年1月19日的第一个对话
  - `session_2025-01-19_02.json` - 2025年1月19日的第二个对话

## 上下文文件内容

每个上下文文件包含以下字段：

| 字段 | 类型 | 说明 |
|------|------|------|
| `session_id` | string | 会话ID，格式为"日期_序号" |
| `topic` | string | 题目名称或类型 |
| `problem_description` | string | 题目描述（简化版） |
| `input_output_example` | string | 样例输入输出 |
| `user_code` | string | 用户提供的代码 |
| `code_language` | string | 代码语言（cpp/python等） |
| `discussion_history` | array | 对话历史记录 |
| `key_points` | array | 关键知识点和讨论要点 |
| `last_update` | string | 最后更新时间（ISO 8601格式） |

## discussion_history 格式

```json
[
  {
    "role": "user",
    "content": "用户的问题或描述",
    "timestamp": "2025-01-19T10:30:00"
  },
  {
    "role": "assistant",
    "content": "智能体的回答",
    "timestamp": "2025-01-19T10:31:00"
  }
]
```

## 智能体工作流程

### 每次回答前

1. 使用 `list_files` 查找 `.codebuddy/context/session_*.json`
2. 判断最新的对话上下文（按日期和序号）
3. 读取上下文内容，查看：
   - 之前讨论的题目
   - 用户代码
   - 讨论历史
   - 关键点

### 判断是否为新对话

新对话的条件：
- 没有上下文文件
- 完全不同的题目/话题
- 用户明确说"换道题"、"新题"等
- 超过2小时未继续同一话题

### 更新上下文时机

- 用户第一次发起新话题 → 创建新上下文
- 用户给出代码 → 更新 `user_code` 和 `code_language`
- 讨论重要知识点 → 添加到 `key_points`
- 每次对话 → 更新 `discussion_history` 和 `last_update`
- 话题明显变化 → 创建新的上下文文件

## 注意事项

1. **必须先读后答** - 每次回答前必须先读取上下文
2. **及时更新** - 每次回答后必须更新上下文
3. **准确记录** - 关键信息（代码、知识点）必须准确记录
4. **合理命名** - 新对话时正确设置 session_id
5. **简洁有效** - 上下文记录要简洁，避免冗余信息
