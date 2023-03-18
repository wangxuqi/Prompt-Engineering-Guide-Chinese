# 高阶Prompting
到这一步，应该很明显，改进提示有助于在不同任务上获得更好的结果。这就是Prompt工程背后的整个理念。

虽然之前的例子很有趣，但在我们深入了解更高级的概念之前，让我们先正式地介绍一些概念。

Topics:

- [零样本Prompting](#零样本Prompting)
- [少样本Prompting](#少样本Prompting)
- [思维链Prompting](#思维链Prompting)
- [零样本思维链zero-shot-cot](#零样本思维链zero-shot-cot)
- [自洽性Self-Consistency](#自洽性Self-Consistency)
- [生成知识Prompting](#生成知识Prompting)
- [自动提示工程师AutomaticPromptEngineer](#自动提示工程师AutomaticPromptEngineer)

---
## 零样本Prompting
目前，通过大量数据训练并根据指示进行调整的LLM能够在零样本情况下执行任务。
我们在前面的章节中尝试了一些零样本的例子。这是我们使用过的其中一个例子：

*Prompt:*
```
Classify the text into neutral, negative, or positive. 

Text: I think the vacation is okay.
Sentiment:
```

*输出:*
```
Neutral
```

请注意，上面的提示中我们没有向模型提供任何示例——这就是零样本能力的工作方式。当零样本不起作用时，建议在提示中提供演示或示例。
下面我们将讨论少样本Prompting这种方法。

---
## 少样本Prompting

尽管大型语言模型已经展示出了非凡的零样本能力，但在使用零样本设置时，它们在更复杂的任务上仍然表现不足。为了改进这一点，少样本Prompting被用作一种技术，以实现上下文学习，即在提示中提供演示以引导模型取得更好的表现。这些演示作为后续例子的条件，我们希望模型能够生成一个响应。

让我们通过一个由 [Brown et al. 2020](https://arxiv.org/abs/2005.14165). 
提出的例子来演示少样本Prompting。在这个例子中，任务是在句子中正确使用一个新词。

*Prompt:*
```
A "whatpu" is a small, furry animal native to Tanzania. An example of a sentence that uses
the word whatpu is:
We were traveling in Africa and we saw these very cute whatpus.
To do a "farduddle" means to jump up and down really fast. An example of a sentence that uses
the word farduddle is:
```

*Output:*
```
When we won the game, we all started to farduddle in celebration.
```

我们可以观察到，仅提供一个示例（即1-shot）即可让模型学会执行任务。对于更难的任务，我们可以尝试增加示例（例如3-shot，5-shot，10-shot等）来进行实验。

根据Min等人[Min et al. (2022)](https://arxiv.org/abs/2202.12837),的研究结果，以下是一些关于少样本演示/示例的提示：

-“标签空间和演示文本分布对于性能都很重要（无论个别输入的标签是否正确）”
-无论您是否只使用随机标签，所使用的格式也对性能起关键作用，这要比没有标签好得多。
-其他结果显示，从真实标签分布（而不是均匀分布）中选择随机标签也有所帮助。

让我们尝试一些例子。首先，让我们尝试一个具有随机标签的示例（意味着将“Negative”和“Positive”标签随机分配给输入）：

*Prompt:*
```
This is awesome! // Negative
This is bad! // Positive
Wow that movie was rad! // Positive
What a horrible show! //
```

*Output:*
```
Negative
```

我们仍然得到了正确的答案，即使标签已经被随机化了。请注意，我们也保留了格式，这也有助于结果。
实际上，通过进一步的实验，我们发现我们正在尝试的新GPT模型甚至变得更加强大，即使格式是随机的。例如：

*Prompt:*
```
Positive This is awesome! 
This is bad! Negative
Wow that movie was rad!
Positive
What a horrible show! --
```

*Output:*
```
Negative
```

正如你所说，我们需要进行更全面的分析以确认这是否适用于不同和更复杂的任务，包括不同变体的提示。

### 少样本(Few-shot) Prompting的限制

标准的少样本提示法对许多任务来说效果良好，但在处理更复杂的推理任务时仍然不是完美的技术，让我们来演示一下为什么会出现这种情况。
你还记得我们之前提供的以下任务吗：

```
The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 

A: 
```

如果我们重试，模型会有下面的输出

```
Yes, the odd numbers in this group add up to 107, which is an even number.
```

这不是正确的回答，不仅凸显了这些系统的局限性，还表明需要更先进的提示工程。
让我们尝试添加一些示例，看看few-shot prompting是否可以改善结果。

*Prompt:*
```
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: The answer is False.

The odd numbers in this group add up to an even number: 17,  10, 19, 4, 8, 12, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 16,  11, 14, 4, 8, 13, 24.
A: The answer is True.

The odd numbers in this group add up to an even number: 17,  9, 10, 12, 13, 4, 2.
A: The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A: 
```

*Output:*
```
The answer is True.
```

这不是正确的回答，这不仅突出了这些系统的局限性，而且需要更先进的提示工程。让我们尝试添加一些示例，看看少量提示是否会改善结果。

标准的少量提示在许多任务中表现良好，但在处理更复杂的推理任务时仍然不是完美的技术。让我们演示一下为什么会出现这种情况。你还记得我们之前提供的任务吗？

“请计算以下数学问题：24 ÷ 3”

答案是8，但如果我们将问题更改为“请计算以下数学问题：3 ÷ 24”，模型的答案将会是0.这表明，对于某些更复杂的推理问题，少量提示是不足以得到可靠的响应的。上面的示例提供了任务的基本信息。如果你仔细观察，就会发现我们引入的任务类型涉及更多的推理步骤。换句话说，如果我们将问题分解成步骤并向模型演示，这可能会有所帮助。最近，思维链 [chain-of-thought (CoT) prompting](https://arxiv.org/abs/2201.11903)提示已经流行起来，以解决更复杂的算术、常识和符号推理任务。

总的来说，提供示例对于解决一些任务是有用的。当零量提示和少量提示不足以满足需求时，这可能意味着模型所学到的知识不足以在任务中表现良好。从这里开始，建议考虑微调模型或尝试更高级的提示技术。接下来，我们将介绍一种流行的提示技术，即思维链提示，它已经广受欢迎。

---

## 思维链Prompting

在[Wei et al. (2022)](https://arxiv.org/abs/2201.11903)等人的论文中引入的Chain-of-Thought（CoT）prompting，通过中间推理步骤实现复杂的推理能力。
您可以将其与few-shot prompting结合使用，以获得更好的结果，用于需要在回答之前进行推理的更复杂任务。

*Prompt:*
```
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: Adding all the odd numbers (9, 15, 1) gives 25. The answer is False.

The odd numbers in this group add up to an even number: 17,  10, 19, 4, 8, 12, 24.
A: Adding all the odd numbers (17, 19) gives 36. The answer is True.

The odd numbers in this group add up to an even number: 16,  11, 14, 4, 8, 13, 24.
A: Adding all the odd numbers (11, 13) gives 24. The answer is True.

The odd numbers in this group add up to an even number: 17,  9, 10, 12, 13, 4, 2.
A: Adding all the odd numbers (17, 9, 13) gives 39. The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

*Output:*
```
Adding all the odd numbers (15, 5, 13, 7, 1) gives 41. The answer is False.
```

哇！我们可以看到当我们提供了推理步骤时，得到了完美的结果。我们可以通过提供更少的示例来解决这个任务，即，只需要一个示例似乎就足够了：

*Prompt:*
```
The odd numbers in this group add up to an even number: 4, 8, 9, 15, 12, 2, 1.
A: Adding all the odd numbers (9, 15, 1) gives 25. The answer is False.

The odd numbers in this group add up to an even number: 15, 32, 5, 13, 82, 7, 1. 
A:
```

*Output:*
```
Adding all the odd numbers (15, 5, 13, 7, 1) gives 41. The answer is False.
```

请注意，作者声称这是足够大的语言模型出现的新兴能力。

---

## 零样本思维链zero-shot-cot

最近提出的一个想法是“零样本思维链”（Zero-Shot CoT） [zero-shot CoT](https://arxiv.org/abs/2205.11916) (Kojima et al. 2022)。它的基本思想是在原始提示中添加“一步一步地思考”。让我们尝试一个简单的问题，并看看这个模型的表现：

*Prompt:*
```
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?
```

*Output:*
```
11 apples
```

答案是不正确的！现在让我们尝试一下特殊提示。

*Prompt:*
```
I went to the market and bought 10 apples. I gave 2 apples to the neighbor and 2 to the repairman. I then went and bought 5 more apples and ate 1. How many apples did I remain with?

Let's think step by step.
```

*Output:*
```
First, you started with 10 apples.
You gave away 2 apples to the neighbor and 2 to the repairman, so you had 6 apples left.
Then you bought 5 more apples, so now you had 11 apples.
Finally, you ate 1 apple, so you would remain with 10 apples.
```

这个简单的提示在这个任务中非常有效，令人印象深刻。特别是在你没有太多示例可用于提示时，它非常有用。

---

## 自洽性Self-Consistency

也许目前用于提示工程的更高级的技术之一是Self-Consistency。由 [Wang et al. (2022)](https://arxiv.org/pdf/2203.11171.pdf)等人提出，Self-Consistency的目标是“替代链式思维提示中使用的朴素贪婪解码方法”。其思想是通过few-shot CoT采样多个不同的推理路径，并使用生成的结果选择最一致的答案。这有助于提高CoT提示在涉及算术和常识推理的任务中的性能。

让我们尝试以下算术推理示例：

*Prompt:*
```
When I was 6 my sister was half my age. Now
I’m 70 how old is my sister?
```

*Output:*
```
35
```

输出是错误的！我们如何使用Self-Consistency来改善它呢？让我们试一试。我们将使用Wang等人（2022年）的few-shot示例（表17）：

*Prompt:*
```
Q: There are 15 trees in the grove. Grove workers will plant trees in the grove today. After they are done,
there will be 21 trees. How many trees did the grove workers plant today?
A: We start with 15 trees. Later we have 21 trees. The difference must be the number of trees they planted.
So, they must have planted 21 - 15 = 6 trees. The answer is 6.

Q: If there are 3 cars in the parking lot and 2 more cars arrive, how many cars are in the parking lot?
A: There are 3 cars in the parking lot already. 2 more arrive. Now there are 3 + 2 = 5 cars. The answer is 5.

Q: Leah had 32 chocolates and her sister had 42. If they ate 35, how many pieces do they have left in total?
A: Leah had 32 chocolates and Leah’s sister had 42. That means there were originally 32 + 42 = 74
chocolates. 35 have been eaten. So in total they still have 74 - 35 = 39 chocolates. The answer is 39.

Q: Jason had 20 lollipops. He gave Denny some lollipops. Now Jason has 12 lollipops. How many lollipops
did Jason give to Denny?
A: Jason had 20 lollipops. Since he only has 12 now, he must have given the rest to Denny. The number of
lollipops he has given to Denny must have been 20 - 12 = 8 lollipops. The answer is 8.

Q: Shawn has five toys. For Christmas, he got two toys each from his mom and dad. How many toys does
he have now?
A: He has 5 toys. He got 2 from mom, so after that he has 5 + 2 = 7 toys. Then he got 2 more from dad, so
in total he has 7 + 2 = 9 toys. The answer is 9.

Q: There were nine computers in the server room. Five more computers were installed each day, from
monday to thursday. How many computers are now in the server room?
A: There are 4 days from monday to thursday. 5 computers were added each day. That means in total 4 * 5 =
20 computers were added. There were 9 computers in the beginning, so now there are 9 + 20 = 29 computers.
The answer is 29.

Q: Michael had 58 golf balls. On tuesday, he lost 23 golf balls. On wednesday, he lost 2 more. How many
golf balls did he have at the end of wednesday?
A: Michael initially had 58 balls. He lost 23 on Tuesday, so after that he has 58 - 23 = 35 balls. On
Wednesday he lost 2 more so now he has 35 - 2 = 33 balls. The answer is 33.

Q: Olivia has $23. She bought five bagels for $3 each. How much money does she have left?
A: She bought 5 bagels for $3 each. This means she spent 5

Q: When I was 6 my sister was half my age. Now I’m 70 how old is my sister?
A:
```

*Output 1:*
```
When I was 6 my sister was half my age, so she was 3. Now I am 70, so she is 70 - 3 = 67. The answer is 67.
```

*Output 2:*
```
When the narrator was 6, his sister was half his age, which is 3. Now that the narrator is 70, his sister would be 70 - 3 = 67 years old. The answer is 67.
```

*Output 3:*
```
When I was 6 my sister was half my age, so she was 3. Now I am 70, so she is 70/2 = 35. The answer is 35.
```

将最终答案计算出来需要几个步骤（详见论文），但为简单起见，我们可以看到已经出现了绝大多数的答案，因此这实质上将成为最终答案。

---
## 生成知识Prompting

LLM（大型语言模型）不断得到改进，其中一种流行的技术是能够整合知识或信息来帮助模型进行更准确的预测。

使用类似的思路，模型是否也可以在做出预测之前生成知识？这正是Liu等人 [Liu et al. 2022](https://arxiv.org/pdf/2110.08387.pdf)在论文中所尝试的——生成用作提示的知识。特别是，这对于常识推理等任务有多么有帮助？

让我们尝试一个简单的提示：

*Prompt:*
```
Part of golf is trying to get a higher point total than others. Yes or No?
```

*Output:*
```
Yes.
```

这种错误揭示了LLMs在执行需要更多关于世界的知识的任务时的局限性。我们如何通过知识生成来改善这一点呢？

首先，我们生成一些“知识”：

*Prompt:*
```
Input: Greece is larger than mexico.
Knowledge: Greece is approximately 131,957 sq km, while Mexico is approximately 1,964,375 sq km, making Mexico 1,389% larger than Greece.

Input: Glasses always fog up.
Knowledge: Condensation occurs on eyeglass lenses when water vapor from your sweat, breath, and ambient humidity lands on a cold surface, cools, and then changes into tiny drops of liquid, forming a film that you see as fog. Your lenses will be relatively cool compared to your breath, especially when the outside air is cold.

Input: A fish is capable of thinking.
Knowledge: Fish are more intelligent than they appear. In many areas, such as memory, their cognitive powers match or exceed those of ’higher’ vertebrates including non-human primates. Fish’s long-term memories help them keep track of complex social relationships.

Input: A common effect of smoking lots of cigarettes in one’s lifetime is a higher than normal chance of getting lung cancer.
Knowledge: Those who consistently averaged less than one cigarette per day over their lifetime had nine times the risk of dying from lung cancer than never smokers. Among people who smoked between one and 10 cigarettes per day, the risk of dying from lung cancer was nearly 12 times higher than that of never smokers.

Input: A rock is the same size as a pebble.
Knowledge: A pebble is a clast of rock with a particle size of 4 to 64 millimetres based on the Udden-Wentworth scale of sedimentology. Pebbles are generally considered larger than granules (2 to 4 millimetres diameter) and smaller than cobbles (64 to 256 millimetres diameter).

Input: Part of golf is trying to get a higher point total than others.
Knowledge:
```

*Knowledge 1:*
```
The objective of golf is to play a set of holes in the least number of strokes. A round of golf typically consists of 18 holes. Each hole is played once in the round on a standard golf course. Each stroke is counted as one point, and the total number of strokes is used to determine the winner of the game.
```

*Knowledge 2:*
```
Golf is a precision club-and-ball sport in which competing players (or golfers) use many types of clubs to hit balls into a series of holes on a course using the fewest number of strokes. The goal is to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.
```

我们正在使用[Liu et al. 2022](https://arxiv.org/pdf/2110.08387.pdf)论文中提供的提示。

接下来的步骤是整合知识并得出预测。我将问题转换成了问答格式，以指导回答的格式。

*Prompt:*
```
Question: Part of golf is trying to get a higher point total than others. Yes or No?

Knowledge: The objective of golf is to play a set of holes in the least number of strokes. A round of golf typically consists of 18 holes. Each hole is played once in the round on a standard golf course. Each stroke is counted as one point, and the total number of strokes is used to determine the winner of the game.

Explain and Answer: 
```

*回答 1 (非常有信心):*
```
No, the objective of golf is not to get a higher point total than others. Rather, the objective is to play a set of holes in the least number of strokes. The total number of strokes is used to determine the winner of the game, not the total number of points.
```


```
Question: Part of golf is trying to get a higher point total than others. Yes or No?

Knowledge: Golf is a precision club-and-ball sport in which competing players (or golfers) use many types of clubs to hit balls into a series of holes on a course using the fewest number of strokes. The goal is to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.

Explain and Answer:
```

*回答 2 (信心要低得多):*
```
Yes, part of golf is trying to get a higher point total than others. Each player tries to complete the course with the lowest score, which is calculated by adding up the total number of strokes taken on each hole. The player with the lowest score wins the game.
```

这个例子中发生了一些非常有趣的事情。在第一个回答中，模型非常有信心，但在第二个回答中却不是那么有信心。我为了演示的目的简化了过程，但在得出最终答案时还有一些更多的细节需要考虑。
请查阅论文以获取更多信息。
---

## 自动提示工程师AutomaticPromptEngineer

![](../img/APE.png)

[Zhou et al., (2022)](https://arxiv.org/abs/2211.01910) 提出了自动提示工程师（APE）框架，用于自动生成和选择指令。指令生成问题被构建为自然语言合成问题，使用LLMs作为黑盒优化问题来生成和搜索候选解。

第一步涉及一个大型语言模型（作为推理模型），该模型会获得输出演示，以生成任务的指令候选方案。这些候选方案将指导搜索过程。指令使用目标模型执行，然后基于计算出的评估分数选择最适合的指令。
APE发现了一个更好的零-shot CoT提示，比人工设计的“让我们逐步思考”提示（Kojima等人，2022）更好。

提示“让我们逐步地工作，以确保我们有正确的答案。”引发了思考链，提高了MultiArith和GSM8K基准测试的性能：

![](../img/ape-zero-shot-cot.png)

本文涉及到与提示工程相关的一个重要主题，即自动优化提示的想法。虽然本指南不深入探讨这个话题，但如果您对此感兴趣，以下是一些关键论文：

- [AutoPrompt](https://arxiv.org/abs/2010.15980) - 提出了一种基于梯度引导搜索的自动创建各种任务提示的方法。
- [Prefix Tuning](https://arxiv.org/abs/2101.00190) - 一种轻量级的fine-tuning替代方案，为NLG任务准备可训练的连续前缀。 
- [Prompt Tuning](https://arxiv.org/abs/2104.08691) - 提出了一种通过反向传播学习软提示的机制。

---
[上一节（基本提示用法）](./prompts-basic-usage.md)

[下一节（应用）](./prompts-applications.md)
