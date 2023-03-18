# Prompt 应用
在这个指南中，我们将介绍一些高级和有趣的方式，通过 prompt engineering 来执行有用的更高级别的任务。


**请注意，本节正在积极开发中。**
主题:
- [生成数据](#生成数据)
- [程序辅助语言模型](#程序辅助语言模型)
- [Python笔记本](#Python笔记本)


---
## 生成数据
LLMs具有生成文本的强大能力。使用有效的提示策略可以引导模型产生更好、更一致、更真实的响应。LLMs也可以特别有用于生成数据，这对于运行各种实验非常有用。例如，我们可以使用它来生成情感分类器的快速样本，如下所示：

*Prompt:*
```
生成10个情感分析的例子。这些例子被分类为正面或负面。生成2个负面例子和8个正面例子。使用以下格式的例子：
问题: <句子>
答案: <情感>
```

*Output:*
```
问题: 我刚刚得到了最好的消息！
答案: 正面

问题: 我们刚刚在工作中得到了加薪！
答案: 正面

问题: 我为今天取得的成就感到非常自豪。
答案: 正面

问题: 我今天过得非常开心！
答案: 正面

问题: 我真的很期待周末。
答案: 正面

问题: 我刚刚得到了最好的礼物！
答案: 正面

问题: 我现在感到非常高兴。
答案: 正面

问题: 我很幸运有这么棒的家人。
答案: 正面

问题: 外面的天气非常阴暗。
答案: 负面

问题: 我刚刚得到了一些可怕的消息。
答案: 负面
```

这非常有用。我们在指南的另一部分中将使用此示例进行不同的测试。

---

## 程序辅助语言模型
 
[Gao et al., (2022)](https://arxiv.org/abs/2211.10435)提出了一种方法，使用LLMs来读取自然语言问题并生成程序作为中间推理步骤。称为程序辅助语言模型（PAL），与思维链提示不同，它不是使用自由形式的文本来获取解决方案，而是将解决步骤卸载到编程运行时，例如Python解释器。
![](../img/pal.png)

让我们通过LangChain和OpenAI GPT-3来看一个例子。我们有兴趣开发一个简单的应用程序，它能够解释所提出的问题并通过利用Python解释器提供答案。

具体来说，我们有兴趣创建一个函数，使得可以使用LLM回答需要日期理解的问题。我们将向LLM提供一个提示，其中包括一些示例，这些示例来自[这里](https://github.com/reasoning-machines/pal/blob/main/pal/prompt/date_understanding_prompt.py)。

这些是我们需要的导入：

```python
import openai
from datetime import datetime
from dateutil.relativedelta import relativedelta
import os
from langchain.llms import OpenAI
from dotenv import load_dotenv
```

我们进行一些少量的配置:

```python
load_dotenv()

# API configuration
openai.api_key = os.getenv("OPENAI_API_KEY")

# for LangChain
os.environ["OPENAI_API_KEY"] = os.getenv("OPENAI_API_KEY")
```

设置模型

```python
llm = OpenAI(model_name='text-davinci-003', temperature=0)
```

设置prompt + question:

```python
question = "Today is 27 February 2023. I was born exactly 25 years ago. What is the date I was born in MM/DD/YYYY?"

DATE_UNDERSTANDING_PROMPT = """
# Q: 2015 is coming in 36 hours. What is the date one week from today in MM/DD/YYYY?
# If 2015 is coming in 36 hours, then today is 36 hours before.
today = datetime(2015, 1, 1) - relativedelta(hours=36)
# One week from today,
one_week_from_today = today + relativedelta(weeks=1)
# The answer formatted with %m/%d/%Y is
one_week_from_today.strftime('%m/%d/%Y')
# Q: The first day of 2019 is a Tuesday, and today is the first Monday of 2019. What is the date today in MM/DD/YYYY?
# If the first day of 2019 is a Tuesday, and today is the first Monday of 2019, then today is 6 days later.
today = datetime(2019, 1, 1) + relativedelta(days=6)
# The answer formatted with %m/%d/%Y is
today.strftime('%m/%d/%Y')
# Q: The concert was scheduled to be on 06/01/1943, but was delayed by one day to today. What is the date 10 days ago in MM/DD/YYYY?
# If the concert was scheduled to be on 06/01/1943, but was delayed by one day to today, then today is one day later.
today = datetime(1943, 6, 1) + relativedelta(days=1)
# 10 days ago,
ten_days_ago = today - relativedelta(days=10)
# The answer formatted with %m/%d/%Y is
ten_days_ago.strftime('%m/%d/%Y')
# Q: It is 4/19/1969 today. What is the date 24 hours later in MM/DD/YYYY?
# It is 4/19/1969 today.
today = datetime(1969, 4, 19)
# 24 hours later,
later = today + relativedelta(hours=24)
# The answer formatted with %m/%d/%Y is
today.strftime('%m/%d/%Y')
# Q: Jane thought today is 3/11/2002, but today is in fact Mar 12, which is 1 day later. What is the date 24 hours later in MM/DD/YYYY?
# If Jane thought today is 3/11/2002, but today is in fact Mar 12, then today is 3/1/2002.
today = datetime(2002, 3, 12)
# 24 hours later,
later = today + relativedelta(hours=24)
# The answer formatted with %m/%d/%Y is
later.strftime('%m/%d/%Y')
# Q: Jane was born on the last day of Feburary in 2001. Today is her 16-year-old birthday. What is the date yesterday in MM/DD/YYYY?
# If Jane was born on the last day of Feburary in 2001 and today is her 16-year-old birthday, then today is 16 years later.
today = datetime(2001, 2, 28) + relativedelta(years=16)
# Yesterday,
yesterday = today - relativedelta(days=1)
# The answer formatted with %m/%d/%Y is
yesterday.strftime('%m/%d/%Y')
# Q: {question}
""".strip() + '\n'
```

```python
llm_out = llm(DATE_UNDERSTANDING_PROMPT.format(question=question))
print(llm_out)
```

```python
exec(llm_out)
print(born)
```

这个程序将输出: `02/27/1998`

---
## Python笔记本

|Description|Notebook|
|--|--|
|Learn how to use the Python interpreter in combination with the language model to solve tasks.|[Program-Aided Language Models](../notebooks/pe-pal.ipynb)|

---

更多示例即将推出！

[上一节（高阶promting）](./prompts-advanced-usage.md)

[下一节（ChatGPT）](./prompts-chatgpt.md)