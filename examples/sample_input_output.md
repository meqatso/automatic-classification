# 语义分组示例

## 示例1：成人内容标签分组

### 输入
```
以我给出的标签、分类集合，给出分组结果：
短视频, 长视频, 直播, 写真, 套图, 动图, 室内, 室外, 特殊场景, 自拍, 专业拍摄, ASMR, 语音聊天, 偷拍, 摆拍
```

### 输出
```json
{
  "input_tags": ["短视频", "长视频", "直播", "写真", "套图", "动图", "室内", "室外", "特殊场景", "自拍", "专业拍摄", "ASMR", "语音聊天", "偷拍", "摆拍"],
  "total_groups": 5,
  "groups": [
    {
      "group_id": 1,
      "representative": "直播",  // 从组成员中选择
      "members": ["短视频", "长视频", "直播"],
      "member_count": 3,
      "selection_reason": "最能代表实时视频内容",
      "selection_confidence": 0.85
    },
    {
      "group_id": 2,
      "representative": "写真",  // 从组成员中选择
      "members": ["写真", "套图", "动图"],
      "member_count": 3,
      "selection_reason": "最具典型性的图片类型",
      "selection_confidence": 0.82
    },
    {
      "group_id": 3,
      "representative": "室内",  // 从组成员中选择
      "members": ["室内", "室外", "特殊场景"],
      "member_count": 3,
      "selection_reason": "最常见的拍摄场景",
      "selection_confidence": 0.88
    },
    {
      "group_id": 4,
      "representative": "ASMR",  // 从组成员中选择
      "members": ["ASMR", "语音聊天"],
      "member_count": 2,
      "selection_reason": "最具特色的音频类型",
      "selection_confidence": 0.75
    },
    {
      "group_id": 5,
      "representative": "自拍",  // 从组成员中选择
      "members": ["自拍", "专业拍摄", "偷拍", "摆拍"],
      "member_count": 4,
      "selection_reason": "最常见和具代表性的拍摄方式",
      "selection_confidence": 0.79
    }
  ],
  "ungrouped_tags": [],
  "statistics": {
    "total_input_tags": 15,
    "grouped_tags": 15,
    "ungrouped_tags": 0,
    "grouping_rate": "100%",
    "average_group_size": 3.0,
    "max_group_size": 4,
    "min_group_size": 2
  }
}
```

## 示例2：混合内容标签分组

### 输入
```
分组这些标签：
美食, 烹饪, 餐厅, 旅游, 旅行, 景点, 电影, 电视剧, 综艺, 音乐, 歌曲, 演唱会
```

### 输出
```json
{
  "input_tags": ["美食", "烹饪", "餐厅", "旅游", "旅行", "景点", "电影", "电视剧", "综艺", "音乐", "歌曲", "演唱会"],
  "total_groups": 4,
  "groups": [
    {
      "group_id": 1,
      "representative": "美食",  // 从组成员中选择
      "members": ["美食", "烹饪", "餐厅"],
      "member_count": 3,
      "selection_reason": "最具代表性和通用性",
      "selection_confidence": 0.87
    },
    {
      "group_id": 2,
      "representative": "旅游",  // 从组成员中选择
      "members": ["旅游", "旅行", "景点"],
      "member_count": 3,
      "selection_reason": "最常见的旅行相关词汇",
      "selection_confidence": 0.89
    },
    {
      "group_id": 3,
      "representative": "电影",  // 从组成员中选择
      "members": ["电影", "电视剧", "综艺"],
      "member_count": 3,
      "selection_reason": "最具典型性的影视形式",
      "selection_confidence": 0.83
    },
    {
      "group_id": 4,
      "representative": "音乐",  // 从组成员中选择
      "members": ["音乐", "歌曲", "演唱会"],
      "member_count": 3,
      "selection_reason": "最基础和通用的音乐相关词汇",
      "selection_confidence": 0.85
    }
  ],
  "ungrouped_tags": [],
  "statistics": {
    "total_input_tags": 12,
    "grouped_tags": 12,
    "ungrouped_tags": 0,
    "grouping_rate": "100%",
    "average_group_size": 3.0,
    "max_group_size": 3,
    "min_group_size": 3
  }
}
```

## 示例3：英文标签分组

### 输入
```
Group these tags:
apple, banana, orange, car, bus, bicycle, computer, laptop, smartphone
```

### 输出
```json
{
  "input_tags": ["apple", "banana", "orange", "car", "bus", "bicycle", "computer", "laptop", "smartphone"],
  "total_groups": 3,
  "groups": [
    {
      "group_id": 1,
      "representative": "apple",  // 从组成员中选择
      "members": ["apple", "banana", "orange"],
      "member_count": 3,
      "selection_reason": "最常见和具代表性的水果",
      "selection_confidence": 0.91
    },
    {
      "group_id": 2,
      "representative": "car",  // 从组成员中选择
      "members": ["car", "bus", "bicycle"],
      "member_count": 3,
      "selection_reason": "最具典型性的交通工具",
      "selection_confidence": 0.86
    },
    {
      "group_id": 3,
      "representative": "computer",  // 从组成员中选择
      "members": ["computer", "laptop", "smartphone"],
      "member_count": 3,
      "selection_reason": "最基础和通用的电子设备",
      "selection_confidence": 0.88
    }
  ],
  "ungrouped_tags": [],
  "statistics": {
    "total_input_tags": 9,
    "grouped_tags": 9,
    "ungrouped_tags": 0,
    "grouping_rate": "100%",
    "average_group_size": 3.0,
    "max_group_size": 3,
    "min_group_size": 3
  }
}
```

## 示例4：包含无法分组标签的情况

### 输入
```
对这些标签进行语义分组：
游泳, 跑步, 篮球, 编程, 代码, 算法, 哲学, 思考, 未知标签XYZ
```

### 输出
```json
{
  "input_tags": ["游泳", "跑步", "篮球", "编程", "代码", "算法", "哲学", "思考", "未知标签XYZ"],
  "total_groups": 3,
  "groups": [
    {
      "group_id": 1,
      "representative": "游泳",  // 从组成员中选择
      "members": ["游泳", "跑步", "篮球"],
      "member_count": 3,
      "selection_reason": "最具代表性的全身运动",
      "selection_confidence": 0.84
    },
    {
      "group_id": 2,
      "representative": "编程",  // 从组成员中选择
      "members": ["编程", "代码", "算法"],
      "member_count": 3,
      "selection_reason": "最基础和核心的计算机技术",
      "selection_confidence": 0.87
    },
    {
      "group_id": 3,
      "representative": "哲学",  // 从组成员中选择
      "members": ["哲学", "思考"],
      "member_count": 2,
      "selection_reason": "更具专业性和代表性",
      "selection_confidence": 0.72
    }
  ],
  "ungrouped_tags": ["未知标签XYZ"],
  "statistics": {
    "total_input_tags": 9,
    "grouped_tags": 8,
    "ungrouped_tags": 1,
    "grouping_rate": "88.9%",
    "average_group_size": 2.67,
    "max_group_size": 3,
    "min_group_size": 2
  }
}
```