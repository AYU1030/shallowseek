# Shallowseek
一个基于扣子平台构建的智能体，不仅能够自动分类邮件、生成专业回复、提炼核心摘要，还支持通过Python代码扩展复杂业务逻辑，并向企业微信实时推送告警消息，实现高度定制化的邮件处理自动化。
# 应用场景
该智能体的核心在于“分类”，主要解决的痛点在于人工处理邮件速度过慢，可能导致客户不满或商机流失，同时减少人力成本，解决人工导致的分类主观性以及因疲劳疏忽邮件中的负面情绪，导致投诉升级的情况；同时传统邮件系统是被动的，团队需要主动查看邮箱才能发现问题，而“摘要生成与推送”可解决重新翻阅冗长邮件导致的事后溯源问题。
1. 企业客服：通过自动分类与回复帮助企业客服处理海量简单客户邮件，减轻人工客服的压力。
2. 销售部门：自动识别咨询类邮件并生成摘要提示进行推送，协助销售人员更好地定位目标客户，并派专人跟进。
3. 个人邮箱管理：帮助商务人士或博主等高邮件流量用户，优先处理紧急或重要的邮件，例如投诉类邮件。
4. 企业管理层：通过推送的摘要，无需翻阅邮件即可掌握客户反馈的核心动态，同时可以快速识别员工提交的各类服务请求。
# 基本结构与核心功能
## （一）主要采用“感知-认知与决策-执行-协同与输出”的结构。
1. 感知：通过“开始”节点接收原始邮件内容 (email_content)。
2. 认知与决策：
（1）核心分类：第一个LLM节点，负责理解邮件内容并进行分类。
（2）逻辑线路：由“选择器”节点构成，根据分类结果，将任务导向不同的回复生成分支。
3. 执行：
（1）回复生成：三个专用的LLM节点，分别针对“咨询”、“投诉”、“其他”场景，生成专业回复草稿。
（2）智能摘要：一个与回复分支并行的LLM节点，专门生成用于推送的精炼摘要。
4. 协同与输出：
（1）消息推送：插件节点，将分类结果和摘要推送至外部协作平台。
（2）结果聚合：“变量聚合”节点，收集各回复分支的结果，确保工作流有统一出口。
（3）最终输出：“结束”节点，将最终的回复草稿返回给用户或下游系统。
## （二）功能详解
1. 智能邮件分类：精准识别邮件意图，分为 【咨询】、【投诉】、【其他】 三类.
2. 专业回复生成：根据邮件类型，自动生成风格匹配、内容专业的回复草稿。
3. 精炼摘要生成：并行提取邮件核心内容，生成摘要。
4. 企业微信实时推送：将分类结果与摘要即时推送到指定企业微信群。
5. Python代码扩展：嵌入Python代码，实现复杂业务逻辑、数据清洗、API调用等。
6. 差异化处理建议：推送消息中包含基于分类的动态处理建议。
7. 结构化数据提取：从非结构化邮件中提取订单号、产品信息等标准化数据。
# 技术架构
<img width="744" height="952" alt="diagram" src="https://github.com/user-attachments/assets/d409293f-e74a-4ebd-90e6-a4c488a85117" />

# 技术栈
1. 核心AI引擎：豆包-Pro系列与Deepseek系列大语言模型
2. 业务逻辑扩展：Python 3.0 运行时环境
3. 工作流编排：扣子平台
4. 消息推送：企业微信机器人Webhook API
5. 外部集成：Requests库支持RESTful API调用

# 具体操作
## 前置要求
1. 扣子平台账号
2. 企业微信账号及群机器人创建权限
3. 需要调用的外部API访问权限
## 安装与配置
1. 企业微信机器人配置
```bash
# 在企业微信群中创建机器人并获取Webhook URL
# 格式: https://qyapi.weixin.qq.com/cgi-bin/webhook/send?key=xxx
```
2. Python代码节点配置

```python
import requests  # HTTP请求
import json      # JSON处理
import re        # 正则表达式
import datetime  # 日期时间
from typing import Dict, List, Optional  # 类型注解
```

3. 环境变量设置

在扣子平台设置敏感信息：

```python
# 在代码节点中通过os.environ读取
api_key = os.environ.get('API_KEY')
database_url = os.environ.get('DATABASE_URL')
```
# 核心模块
1. 邮件分类模块
（1）位置：首个大模型节点
（2）输入：原始邮件内容
（3）输出：咨询 / 投诉 / 其他
（4）特点：基于多级规则和关键词的精准分类

3. Python扩展模块
   
模式A：数据清洗

```python
def main(email_content: str) -> str:
    # 清理邮件签名、标准化格式
    cleaned = " ".join(email_content.split())
    return cleaned
```

模式B：API集成

```python
def main(product_name: str) -> Dict:
    # 调用库存API
    response = requests.get(f"https://api.example.com/stock/{product_name}")
    return response.json()
```

模式C：业务规则引擎

```python
def main(content: str, category: str) -> Dict:
    # 计算优先级和分配负责人
    urgency = "高" if "紧急" in content else "中" if category == "投诉" else "低"
    return {"urgency": urgency, "owner": "客服主管" if urgency == "高" else "客服专员"}
```

模式D：数据提取

```python
def main(email_content: str) -> Dict:
    # 提取结构化信息
    order_match = re.search(r'订单[号|码][:：]?\s*(\w+)', email_content)
    return {"order_id": order_match.group(1) if order_match else None}
```

3. 消息推送模块

模板：

```markdown
·分类：{{大模型.output}}
·摘要：{{摘要生成.output}}
·建议：{{Python节点.output.urgency}} - 分配给{{Python节点.output.owner}}
```
# 测试方案

## 测试邮件集

主要分为以下几类：
1. 标准咨询/投诉/其他邮件
2. 边界案例（咨询中带负面反馈）
3. 压力测试（极短邮件、情绪激烈邮件）

## 自动化测试

```python
# 在Python节点中可集成测试逻辑
def test_classification():
    test_cases = [
        ("这个产品多少钱？", "咨询"),
        ("质量太差要求退款！", "投诉"),
        ("感谢你们的服务", "其他")
    ]
    # 自动化验证分类准确性
```

# 生产环境配置

1. 性能优化
   · 设置LLM温度参数：0.1-0.3
   · 配置代码执行超时：30秒
   · 启用工作流缓存
2. 监控告警
   · 企业微信推送失败告警
   · API调用成功率监控
   · 工作流执行时长监控
3. 错误处理

```python
try:
    # 业务逻辑
    result = process_email(content)
except Exception as e:
    # 优雅降级
    logger.error(f"处理失败: {str(e)}")
    return {"status": "error", "message": "系统繁忙，请稍后重试"}
```
# 改进策略

## 优化思路

1. 数据驱动优化：收集分类错误案例，优化提示词
2. AB测试：对比不同回复模板的转化率
3. 用户反馈：集成满意度评分，持续改进

## 拓展参考

1. 知识库集成
2. 多语言支持
3. 邮件附件解析
4. 智能路由到不同部门

## 设计与执行过程中的困难
由于企业微信添加外部人员的权限问题，导致无法创建群聊，因此省略导入机器人的步骤，采用Python代码来进行一定弥补；经过多次测试，豆包pro系列和Deepseek系列大模型相较其他大模型而言更加准确，但是仍偶尔会出现分类错误的情况。
