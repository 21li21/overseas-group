# 项目成员分工详表

## 角色与成员

### 项目总控 (组长)
- **成员**: 李婧涵
- **核心职责明细**:
  - 制定每日进度表与Deadline
  - 审核技术方案与LaTeX样式框架
  - 协调跨组协作
  - 汇总GitHub提交物 (assignment1.md+PDF)
  - 汇报并演练讲解逻辑

### 技术实现组 
- **成员**: 钟圣曦
- **核心职责明细**:
  - 负责舆情分析中的LLM接口调用 (如ChatGPT API)
  - 编写数据爬取/情感分析的核心脚本
  - 验证不同提示词对报告生成效果的影响
  - 输出技术说明文档 (含接口参数、代码片段、报错解决方案)

### 技术实现组 
- **成员**: 郭梓良
- **核心职责明细**:
  - 定制LaTeX模板 (标题样式/图表编号/参考文献格式)
  - 开发自动排版工具 (Python+LaTeX联动)
  - 解决图表嵌入、公式渲染的技术难题
  - 为文档组提供LaTeX使用手册

### 文档生成组 
- **成员**: 刘星雨
- **核心职责明细**:
  - 根据大纲撰写舆情报告正文 (背景/数据/结论等章节)
  - 协同技术组插入动态生成内容 (自动更新数据图表)
  - 维护Markdown主文档 (assignment1.md) 中的过程性记录
  - 初稿版本管理

### 文档生成组
- **成员**: 郭梓良
- **核心职责明细**:
  - 将Markdown内容转换为LaTeX格式
  - 优化排版细节 (行间距/字体/页眉页脚)
  - 嵌入技术组提供的动态脚本 (如Python生成的图表)
  - 处理交叉引用和目录生成
    
### 文档校对组
- **成员**: 田嘉铭
- **核心职责明细**:
   - 负责准备 `assignment1.md` 文件，内容包括介绍所有用到的提示词、用到的代码、latex 脚本以及设计思路等，并提供必要的截图。
   - 编写 `assignment1.md` 文件，确保内容详尽。
   - 提供相关的截图和示例。

### 文档校对组
- **成员**: 张君仪
- **核心职责明细**:
   - 负责 `assignment1.md` 文件的章节安排。
   - 确保 `assignment1.md` 文件包含以下章节：
   - 舆情报告样式检查清单（简要说明报告结构和样式，通过审查帮助成员将样式的理解拉齐）
   - 具体展开的内容：各个步骤用到的提示词、所用到的脚本、所生成的 latex 代码等等，可以适当加入图片以清楚介绍使用的平台和工具。
 



# 舆情报告样式检查清单

## ▍报告结构规范

### (1) 基础模块

| 章节 | 要求 | 样例参考 |
|------|------|----------|
| **封面** | 包含主副标题+日期+团队标识 | "胖猫事件多平台舆情分析报告 - 第3组" |
| **目录** | 自动生成带页码目录（推荐LaTeX实现） | 如报告中3-13节分层架构 |
| **附录** | 含完整数据表格/模型详情（非必要不放置正文） | 原始数据集/分词词典 |

### (2) 核心章节

```plaintext
**1. 引言（背景与研究意义）**
**2. 摘要（核心结论需加粗标红）**  
**3. 时间线与关键节点（配合流程图）**  
**4. 数据与方法（含爬虫/情感分析技术路线）**  
**5. 平台差异分析（微博/抖音对比需分节）**  
**6. 危机公关案例（事件相关的品牌影响）**  
**7. 讨论与启示（分条目提出建议）**  
**8. 参考文献（遵循APA格式）**
```




# 舆情分析项目技术文档

## 一、提示词库

### 数据采集提示词

```python
search_query = {
    "platform": "weibo",
    "keywords": ["胖猫事件", "重庆跳江", "情感操控"],
    "time_range": ("2024-04-11", "2024-05-19"),
    "interaction_threshold": 300  # 只采集点赞≥300的帖子
}
```

### 情感分析提示词（LLM）

```plaintext
PROMPT = """基于以下微博文本进行细粒度情感评分:
文本内容: {text}

评分规则:
- 情绪强度: 正面(0-20)/中性(0)/负面(-10-0)
- 标注维度: 
  愤怒值(0-10) 悲伤值(0-10) 
  是否含网暴内容(是/否)
  关键实体识别(如[胖猫]、[谭某])

用JSON格式返回分析结果"""
```

## 二、代码实现

### 数据采集核心代码

```python
# weibo_crawler.py
import snscrape.modules.weibo as s_weibo
import pandas as pd

def fetch_weibo_data(keywords, start_date, end_date):
    query = ' OR '.join(keywords) + f' since:{start_date} until:{end_date}'
    tweets = []
    for i, tweet in enumerate(s_weibo.WeiboSearchScraper(query).get_items()):
        if tweet.likeCount >= 300:
            tweets.append({
                'date': tweet.date.strftime("%Y-%m-%d"),
                'content': tweet.content,
                'likes': tweet.likeCount,
                'reposts': tweet.repostCount
            })
        if i >= 10000:  # 限制最大采集量
            break
    return pd.DataFrame(tweets)
```

### 情感分析流水线

```python
# sentiment_analysis.py
from transformers import pipeline

class EmotionAnalyzer:
    def __init__(self):
        self.classifier = pipeline(
            "text-classification", 
            model="uer/roberta-base-finetuned-jd-binary-chinese",
            tokenizer="uer/roberta-base-finetuned-jd-binary-chinese"
        )
    
    def analyze_text(self, text):
        result = self.classifier(text)
        anger_level = 1 if '愤怒' in text else 0
        return {
            'sentiment': result[0]['label'],
            'confidence': result[0]['score'],
            'anger_level': anger_level
        }
```

## 三、LaTeX 自动化系统

### 报告模板结构

```latex
% report_template.tex
\documentclass[12pt]{article}
\usepackage[UTF8]{ctex}
\usepackage{graphicx}

\begin{document}
\title{\textbf{<< title >>}}
\author{<< author >>}
\date{<< date >>}
\maketitle

\section*{摘要}
<< abstract >>

\section{数据分析}
\begin{figure}[htbp]
  \centering
  \includegraphics[width=0.8\textwidth]{<< graph_path >>}
  \caption{舆情趋势时序图}
\end{figure}

<< include_sections >>
\end{document}
```

### Markdown 转 LaTeX 脚本

```bash
#!/bin/bash
# convert.sh
pandoc report.md \
  --template=template.tex \
  --pdf-engine=xelatex \
  -V mainfont="Microsoft YaHei" \
  -V geometry:"top=2cm, bottom=2cm, left=3cm, right=3cm" \
  -o output.pdf
```

## 四、系统设计架构

### 技术路线图

```mermaid
graph TD
    A[数据采集层] -->|微博爬虫| B(原始数据集)
    A -->|Sina Opinion工具| B
    B --> C[处理分析层]
    C --> D{情感分析模型}
    C --> E[时序趋势分析]
    C --> F[关键词提取]
    D --> G[情感可视化]
    E --> G
    F --> G
    G --> H[报告生成层]
    H -->|LaTeX编译| I(最终PDF报告)
```

### 关键设计亮点

- 动态模板系统：通过占位符实现内容热替换
- 混合分析策略：深度学习模型 + 规则引擎提升准确率
- 自动样式适应：根据内容长度动态调整图表尺寸

## 五、运行环境要求

- python: 3.9+
- dependencies:
  - snscrape>=0.4.3
  - transformers[torch]==4.26.0
  - pandas>=1.4.0
- latex:
  - texlive-full
  - pandoc>=2.14



