# 实现指南

## 核心算法思路

### 1. 语义相似度计算
- 使用预训练的词向量模型（如Word2Vec、BERT等）
- 将每个标签转换为向量表示
- 计算标签之间的余弦相似度

### 2. 聚类分组算法
- **DBSCAN**：基于密度的聚类，适合自动确定簇数量
- **K-means**：需要预先指定簇数量
- **层次聚类**：生成树状结构，可动态切割

### 3. 组内代表性元素选择（关键要求）
**重要约束**：representative必须是该组members中的一个原始元素，不能是新生成的标签！

选择方法：
- **中心点法**：选择距离簇中心最近的原始标签
- **平均相似度法**：选择与组内其他成员平均相似度最高的原始标签
- **TF-IDF法**：选择在簇内最具代表性、簇外最不常见的原始标签
- **语义典型性法**：选择最具典型意义的原始标签
- **频率优先法**：如果可用，选择最常见或最通用的原始标签

## Python实现示例

```python
import json
import re
from typing import List, Dict, Any
import numpy as np
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.metrics.pairwise import cosine_similarity
from sklearn.cluster import DBSCAN

class SemanticGrouper:
    def __init__(self, config_path: str = None):
        self.config = self._load_config(config_path)
        self.vectorizer = TfidfVectorizer()
        
    def _load_config(self, config_path: str) -> Dict:
        """加载配置"""
        if config_path:
            with open(config_path, 'r', encoding='utf-8') as f:
                return json.load(f)
        return {
            'similarity_threshold': 0.6,
            'min_group_size': 2,
            'max_groups': 10,
            'enable_synonym_detection': True
        }
    
    def parse_input(self, input_text: str) -> List[str]:
        """解析输入文本，提取标签"""
        # 移除触发词和说明文字
        patterns = [
            r'以我给出的标签、分类集合，给出分组结果[:：]\s*',
            r'分组这些标签[:：]\s*',
            r'对这些标签进行语义分组[:：]\s*',
            r'Group these tags[:：]\s*'
        ]
        
        for pattern in patterns:
            input_text = re.sub(pattern, '', input_text, flags=re.IGNORECASE)
        
        # 分割标签（支持逗号、空格、换行、分号等分隔符）
        tags = re.split(r'[,，\s;\n]+', input_text.strip())
        
        # 清理空标签和空白字符
        tags = [tag.strip() for tag in tags if tag.strip()]
        
        return tags
    
    def calculate_similarity_matrix(self, tags: List[str]) -> np.ndarray:
        """计算标签相似度矩阵"""
        # 使用TF-IDF向量化
        tfidf_matrix = self.vectorizer.fit_transform(tags)
        
        # 计算余弦相似度
        similarity_matrix = cosine_similarity(tfidf_matrix)
        
        return similarity_matrix
    
    def cluster_tags(self, tags: List[str], similarity_matrix: np.ndarray) -> List[int]:
        """对标签进行聚类"""
        # 使用DBSCAN聚类
        # 将相似度转换为距离：距离 = 1 - 相似度
        distance_matrix = 1 - similarity_matrix
        
        # DBSCAN参数
        eps = 1 - self.config['similarity_threshold']
        min_samples = self.config['min_group_size']
        
        dbscan = DBSCAN(eps=eps, min_samples=min_samples, metric='precomputed')
        labels = dbscan.fit_predict(distance_matrix)
        
        return labels
    
    def select_representative(self, group_tags: List[str], similarity_matrix: np.ndarray, 
                            tag_indices: List[int]) -> Dict[str, Any]:
        """为每个组选择代表性标签（从原始组成员中选择）"""
        if not group_tags:
            return {"representative": "", "reason": "空组"}
        
        # 方法1：选择与其他标签平均相似度最高的原始标签
        group_similarity_matrix = similarity_matrix[np.ix_(tag_indices, tag_indices)]
        avg_similarities = group_similarity_matrix.mean(axis=1)
        
        # 找到平均相似度最高的原始标签
        best_idx = np.argmax(avg_similarities)
        representative = group_tags[best_idx]
        
        # 生成选择理由
        reasons = [
            "与组内其他成员平均相似度最高",
            "最具语义中心性",
            "最能代表该组特征",
            "最具典型性和通用性",
            "语义上最接近组中心"
        ]
        
        return {
            "representative": representative,
            "reason": reasons[best_idx % len(reasons)],
            "confidence": float(avg_similarities[best_idx])
        }
    
    def group_tags(self, input_text: str) -> Dict[str, Any]:
        """主函数：对标签进行分组"""
        # 1. 解析输入
        tags = self.parse_input(input_text)
        
        if not tags:
            return {
                "error": "未找到有效的标签",
                "input_text": input_text
            }
        
        # 2. 计算相似度矩阵
        similarity_matrix = self.calculate_similarity_matrix(tags)
        
        # 3. 聚类
        cluster_labels = self.cluster_tags(tags, similarity_matrix)
        
        # 4. 组织分组结果
        groups = []
        ungrouped_tags = []
        
        # 获取所有簇（排除噪声点，即label=-1）
        unique_labels = set(cluster_labels)
        
        for label in unique_labels:
            if label == -1:
                # 噪声点，无法分组
                noise_indices = np.where(cluster_labels == label)[0]
                for idx in noise_indices:
                    ungrouped_tags.append(tags[idx])
            else:
                # 有效分组
                group_indices = np.where(cluster_labels == label)[0]
                group_tags = [tags[i] for i in group_indices]
                
                # 选择代表性标签（从原始组成员中选择）
                selection_result = self.select_representative(
                    group_tags, similarity_matrix, group_indices.tolist()
                )
                
                groups.append({
                    "group_id": len(groups) + 1,
                    "representative": selection_result["representative"],  # 必须是group_tags中的一个
                    "members": group_tags,
                    "member_count": len(group_tags),
                    "selection_reason": selection_result["reason"],
                    "selection_confidence": selection_result["confidence"]
                })
        
        # 5. 生成最终结果
        result = {
            "input_tags": tags,
            "total_groups": len(groups),
            "groups": groups,
            "ungrouped_tags": ungrouped_tags,
            "statistics": {
                "total_input_tags": len(tags),
                "grouped_tags": len(tags) - len(ungrouped_tags),
                "ungrouped_tags": len(ungrouped_tags),
                "grouping_rate": f"{(len(tags) - len(ungrouped_tags)) / len(tags) * 100:.1f}%"
            }
        }
        
        return result
    
    def format_output(self, result: Dict[str, Any]) -> str:
        """格式化输出为JSON字符串"""
        return json.dumps(result, ensure_ascii=False, indent=2)

# 使用示例
if __name__ == "__main__":
    grouper = SemanticGrouper()
    
    # 测试输入
    test_input = """
    以我给出的标签、分类集合，给出分组结果：
    短视频, 长视频, 直播, 写真, 套图, 动图, 室内, 室外, 特殊场景
    """
    
    result = grouper.group_tags(test_input)
    output = grouper.format_output(result)
    
    print(output)
```

## 高级优化建议

### 1. 使用预训练模型
- 中文：BERT、ERNIE、RoBERTa
- 英文：BERT、GPT embeddings、Sentence-BERT

### 2. 同义词处理
- 集成同义词词典
- 使用WordNet（英文）或HowNet（中文）
- 基于上下文的同义词识别

### 3. 多语言支持
- 检测输入语言
- 使用多语言BERT模型
- 语言特定的预处理

### 4. 性能优化
- 缓存词向量
- 批量处理
- 异步计算

### 5. 可配置性
- 动态调整相似度阈值
- 支持自定义分组规则
- 可扩展的插件架构

## 集成到OpenClaw技能

在OpenClaw技能中，你可以：

1. 创建一个主处理脚本
2. 在技能触发时调用分组逻辑
3. 将结果格式化为JSON输出
4. 提供错误处理和用户反馈

## 测试策略

1. **单元测试**：测试各个组件功能
2. **集成测试**：测试完整工作流程
3. **性能测试**：测试处理大量标签的能力
4. **用户体验测试**：确保输出格式友好易用