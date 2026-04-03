---
name: automatic-classification
version: 1.0.0
description: >
  将用户给出的标签、分类集合按意思相近进行智能分组，并从每个组的成员中选择一个最具代表性的原始元素作为该组的代表。
  关键规则：representative必须是组成员中的一个原始标签，不能是新创建的概括性标签。
  Use when: 用户需要将标签集合按语义相似度分组，或说"以我给出的标签、分类集合，给出分组结果"。
author: ShiQing
homepage: https://github.com/meqatso/automatic-classification.git
license: MIT
---

# 语义相似度自动分组

## 功能描述

本技能用于对用户提供的任意标签、分类集合进行智能语义分析，将意思相近的元素自动分组，并为每个组选择一个最具代表性的元素作为组名。

## 核心能力

1. **语义相似度分析**：识别标签之间的语义关联
2. **智能分组**：将相似标签自动归为一组
3. **组内代表性选择**：从每个组的成员中选择一个最具代表性的元素作为该组的代表
4. **JSON输出**：生成结构化的分组结果，便于开发使用

## 工作流程

### 1. 接收输入
- 获取用户提供的标签、分类字符串集合
- 支持逗号、空格、换行等多种分隔符
- 如果用户未提供任何标签，提示："请提供需要分组的标签或分类集合"

### 2. 预处理
- 清理空白字符和特殊符号
- 识别中英文标签
- 标准化标签格式

### 3. 语义分析与分组
- 使用语义相似度算法分析标签关系
- 将相似度高的标签归为同一组
- 自动确定最佳分组数量

### 4. 组内代表性元素选择
- **从组成员中选择**：representative字段必须是该组members中的一个元素
- **选择标准**：
  - **语义中心性**：与组内其他成员语义最接近
  - **通用性**：最能代表该组特征
  - **简洁性**：名称清晰简洁
  - **典型性**：最具典型意义的元素

### 5. 生成输出
- 将结构化的json串形式的分组结果直接给到用户
- 包含分组信息、代表性元素、组成员等

## 输出格式

```json
{
  "input_tags": ["标签1", "标签2", "标签3", ...],
  "total_groups": 3,
  "groups": [
    {
      "group_id": 1,
      "representative": "标签2",  // 注意：representative必须是members中的一个元素
      "members": ["标签1", "标签2", "标签3"],
      "member_count": 3,
      "selection_reason": "与组内其他成员平均相似度最高"
    },
    {
      "group_id": 2,
      "representative": "标签4",  // 注意：representative必须是members中的一个元素
      "members": ["标签4", "标签5"],
      "member_count": 2,
      "selection_reason": "最具通用性和代表性"
    }
  ],
  "ungrouped_tags": ["无法归类的标签1", "无法归类的标签2"],
  "statistics": {
    "total_input_tags": 10,
    "grouped_tags": 8,
    "ungrouped_tags": 2,
    "grouping_rate": "80%"
  }
}
```

## 使用示例

### 示例1：成人内容标签分组
用户输入：
```
以我给出的标签、分类集合，给出分组结果：
短视频, 长视频, 直播, 写真, 套图, 动图, 室内, 室外, 特殊场景, 自拍, 专业拍摄
```

预期输出：
```json
{
  "input_tags": ["短视频", "长视频", "直播", "写真", "套图", "动图", "室内", "室外", "特殊场景", "自拍", "专业拍摄"],
  "total_groups": 4,
  "groups": [
    {
      "group_id": 1,
      "representative": "直播",  // 从组成员中选择：短视频、长视频、直播
      "members": ["短视频", "长视频", "直播"],
      "member_count": 3,
      "selection_reason": "最能代表视频内容多样性"
    },
    {
      "group_id": 2,
      "representative": "写真",  // 从组成员中选择：写真、套图、动图
      "members": ["写真", "套图", "动图"],
      "member_count": 3,
      "selection_reason": "最具典型性和通用性"
    },
    {
      "group_id": 3,
      "representative": "室内",  // 从组成员中选择：室内、室外、特殊场景
      "members": ["室内", "室外", "特殊场景"],
      "member_count": 3,
      "selection_reason": "最常见和具代表性的拍摄场景"
    },
    {
      "group_id": 4,
      "representative": "自拍",  // 从组成员中选择：自拍、专业拍摄
      "members": ["自拍", "专业拍摄"],
      "member_count": 2,
      "selection_reason": "更常见和具代表性的拍摄方式"
    }
  ],
  "ungrouped_tags": [],
  "statistics": {
    "total_input_tags": 11,
    "grouped_tags": 11,
    "ungrouped_tags": 0,
    "grouping_rate": "100%"
  }
}
```

### 示例2：通用标签分组
用户输入：
```
分组这些标签：苹果, 香蕉, 橙子, 汽车, 公交车, 自行车, 北京, 上海, 广州
```

预期输出：
```json
{
  "input_tags": ["苹果", "香蕉", "橙子", "汽车", "公交车", "自行车", "北京", "上海", "广州"],
  "total_groups": 3,
  "groups": [
    {
      "group_id": 1,
      "representative": "苹果",  // 从组成员中选择：苹果、香蕉、橙子
      "members": ["苹果", "香蕉", "橙子"],
      "member_count": 3,
      "selection_reason": "最常见和具代表性的水果"
    },
    {
      "group_id": 2,
      "representative": "汽车",  // 从组成员中选择：汽车、公交车、自行车
      "members": ["汽车", "公交车", "自行车"],
      "member_count": 3,
      "selection_reason": "最具典型性的交通工具"
    },
    {
      "group_id": 3,
      "representative": "北京",  // 从组成员中选择：北京、上海、广州
      "members": ["北京", "上海", "广州"],
      "member_count": 3,
      "selection_reason": "首都，最具代表性"
    }
  ],
  "ungrouped_tags": [],
  "statistics": {
    "total_input_tags": 9,
    "grouped_tags": 9,
    "ungrouped_tags": 0,
    "grouping_rate": "100%"
  }
}
```

## 实现方式

本技能使用以下方法实现语义分组：

1. **词向量模型**：将标签转换为向量表示
2. **相似度计算**：使用余弦相似度等度量方法
3. **聚类算法**：可选DBSCAN、K-means等聚类方法
4. **代表性选择**：基于向量中心或TF-IDF选择代表性标签

## 配置选项

可在`references/config.json`中配置：
- 相似度阈值
- 最小分组大小
- 最大分组数量
- 语言偏好（中文/英文）

## 重要规则

### 代表性元素选择规则
1. **必须从组成员中选择**：`representative`字段的值必须是该组`members`数组中的一个原始元素
2. **不能创建新标签**：禁止生成新的概括性标签（如"视频内容"、"图片内容"等）
3. **选择依据**：基于语义中心性、典型性、通用性等标准从原始成员中选择
4. **提供选择理由**：在`selection_reason`字段中说明为什么选择该元素作为代表

### 其他注意事项
1. 支持中英文混合标签处理
2. 自动处理同义词和近义词
3. 可配置分组粒度
4. 输出格式为开发友好的JSON结构
5. 对于无法明确分组的标签，保留在`ungrouped_tags`中

## 触发词

- "以我给出的标签、分类集合，给出分组结果"
- "将这些标签按意思相近分组"
- "自动分类这些标签"
- "语义分组"
- "标签聚类"
