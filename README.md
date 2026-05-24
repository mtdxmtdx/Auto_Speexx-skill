# Speexx Exercise Automation Skill

Claude Code 自动化完成 Speexx 语言学习练习的 Agent Skill。

## 功能

- 自动完成所有题型：拖拽填空、表格配对、选择题、文本填空、连词成句
- 每道练习确保 100 分后才进入下一题
- 支持发音练习自动完成（可选）
- 自动跳过视频练习
- 可指定完成小节数量或范围
- 等级测试小节默认跳过；如用户要求完成，直接推断并填写答案（无需 Correction 验证）

## 安装

```bash
git clone https://github.com/mtdxmtdx/Auto_Speexx-skill.git ~/.claude/skills/speexx-exercise-automation
```

## 前置条件

- Chrome DevTools MCP server 已启动并连接
- 已登录 Speexx 账号

## 使用

在 Claude Code 中输入 `/speexx-exercise-automation` 触发 skill，然后按提示回答：

1. 打开需要完成的第一个练习页面
2. 需要完成多少小节？（如 6 小节，或 12-14 小节）
3. 是否需要完成发音练习？
4. 确认是否跳过等级测试

## 支持的题型

| 题型 | 说明 |
|------|------|
| 拖拽填空 | jQuery UI drag-drop，自动放置词语到正确位置 |
| 表格配对 | 交换错位单元格完成配对 |
| 选择/单选 | 自动勾选正确选项 |
| 文本填空 | 在输入框中填入正确单词 |
| 连词成句 | 将乱序单词排列成正确句子 |
| 视频 | 直接跳过 |
| 发音 | 触发麦克风，检测反馈后跳过 |

## 文件结构

```
speexx-exercise-automation/
├── SKILL.md                        # Skill 定义与执行流程
├── README.md                       # 本文件
└── references/
    └── exercise-patterns.md        # 各题型 JavaScript 自动化代码
```
