# Lecture Transcript Skill

将PPT课件转换为大学老师讲课风格的口语化逐字稿。

## 安装

在项目目录下执行：

```bash
mkdir -p .claude/skills
git clone https://github.com/AlbertYang0801/lecture-transcript-skill.git .claude/skills/lecture-transcript-skill
```

## 使用

在 Claude Code 中输入：

```
/lecture-transcript
```

然后提供你的PPT内容。Claude会：
1. 自动检测章节结构，显示章节列表
2. 询问是否并行执行或顺序处理
3. 按章节生成口语化逐字稿
4. 合并所有章节并输出最终文档

## 功能特点

- 将PPT定义转换为口语化讲解
- 用生活例子引入抽象概念
- 保持课堂互动感（"同学们"、"大家想想"）
- **并行处理**：多章节可并行执行，提升效率
- **任务追踪**：使用Task工具管理章节处理进度
- **失败重试**：自动重试2次，合并时提示用户处理失败章节
- **风格检查**：合并Agent检查一致性和完整性

## 示例

输入PPT内容后，输出效果：

```
**p12 插入排序基本思想**

同学们好。今天我们学习插入排序。

插入排序的思想其实很好理解，大家在生活中都用过。想想你打扑克牌的时候，每摸一张牌，你都会把它插到手里已有的牌中合适的位置——小的插左边，大的插右边，手里的牌始终保持有序。这就是插入排序！
```

## License

MIT