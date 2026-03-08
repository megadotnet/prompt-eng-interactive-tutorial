# 第 3 章：分配角色（角色提示）

- [课程](#课程)
- [练习](#练习)
- [示例游乐场](#example-playground)

## 设置

运行以下设置单元来加载您的 API 密钥并建立“get_completion”辅助函数。

```python
!pip install anthropic

# Import python's built-in regular expression library
import re
import anthropic

# Retrieve the API_KEY & MODEL_NAME variables from the IPython store
%store -r API_KEY
%store -r MODEL_NAME

client = anthropic.Anthropic(api_key=API_KEY)

def get_completion(prompt: str, system_prompt=""):
    message = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        system=system_prompt,
        messages=[
          {"role": "user", "content": prompt}
        ]
    )
    return message.content[0].text
```

---

## 课程

继续讨论克劳德除了你所说的之外没有任何背景的主题，有时**提示克劳德扮演一个特定的角色（包括所有必要的背景）**很重要。这也称为角色提示。角色背景越详细越好。

**为克劳德赋予角色可以提高克劳德在各个领域的表现**，从写作到编码再到总结。这就像当人类被告知“像______一样思考”时有时会得到帮助。角色提示还可以改变克劳德回应的风格、语气和方式。

**注意：** 角色提示可以在系统提示中发生，也可以作为用户消息轮流的一部分发生。

### 示例

在下面的示例中，我们看到，在没有角色提示的情况下，当要求克劳德给出关于滑板运动的单句观点时，他提供了**直接且非程式化的答案**。

然而，当我们让克劳德扮演猫的角色时，克劳德的视角发生了变化，因此**克劳德的反应语气、风格、内容都会适应新的角色**。

**注意：** 您可以使用的一项额外技巧是 **为 Claude 的目标受众提供背景信息**。下面，我们可以调整提示来告诉 Claude 它应该与谁交谈。“你是一只猫”与“你是一只猫在和一群滑板手说话”产生的反应截然不同。

下面是系统提示中没有角色提示的提示：

```python
# Prompt
PROMPT = "In one sentence, what do you think about skateboarding?"

# Print Claude's response
print(get_completion(PROMPT))
```

这是相同的用户问题，除了角色提示之外。

```python
# System prompt
SYSTEM_PROMPT = "You are a cat."

# Prompt
PROMPT = "In one sentence, what do you think about skateboarding?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

您可以使用角色提示来让克劳德模仿某些写作风格、用某种声音说话或引导其答案的复杂性。**角色提示还可以使克劳德更好地执行数学或逻辑任务。**

例如，在下面的例子中，有一个明确的正确答案，那就是“是”。然而，克劳德错了，认为它缺乏信息，但事实并非如此：

```python
# Prompt
PROMPT = "Jack is looking at Anne. Anne is looking at George. Jack is married, George is not, and we don’t know if Anne is married. Is a married person looking at an unmarried person?"

# Print Claude's response
print(get_completion(PROMPT))
```

现在，如果我们**让 Claude 充当逻辑机器人**会怎么样？这将如何改变克劳德的答案？

事实证明，克劳德的新角色分配是正确的。（尽管显然并非出于所有正确的原因）

```python
# System prompt
SYSTEM_PROMPT = "You are a logic bot designed to answer complex logic problems."

# Prompt
PROMPT = "Jack is looking at Anne. Anne is looking at George. Jack is married, George is not, and we don’t know if Anne is married. Is a married person looking at an unmarried person?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

**注意：**您将在本课程中学到的是，**您可以使用许多即时工程技术来得出类似的结果**。您使用哪种技术取决于您和您的偏好！我们鼓励您**通过实验找到适合自己的快速工程风格**。

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 3.1 - 数学校正](#exercise-31---数学校正)

### 练习 3.1 - 数学修正
在某些情况下，**克劳德可能会在数学方面遇到困难**，甚至是简单的数学。下面，克劳德错误地将数学问题评估为正确解决，尽管第二步中存在明显的算术错误。请注意，克劳德实际上在逐步执行时发现了错误，但并没有立即得出整体解决方案错误的结论。

修改“PROMPT”和/或“SYSTEM_PROMPT”，使 Claude 将解决方案评分为“错误解决”，而不是“正确解决”。

```python
# System prompt - if you don't want to use a system prompt, you can leave this variable set to an empty string
SYSTEM_PROMPT = ""

# Prompt
PROMPT = """Is this equation solved correctly below?

2x - 3 = 9
2x = 6
x = 3"""

# Get Claude's response
response = get_completion(PROMPT, SYSTEM_PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    if "incorrect" in text or "not correct" in text.lower():
        return True
    else:
        return False

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_3_1_hint; print(exercise_3_1_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Prompt
PROMPT = "In one sentence, what do you think about skateboarding?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# System prompt
SYSTEM_PROMPT = "You are a cat."

# Prompt
PROMPT = "In one sentence, what do you think about skateboarding?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

```python
# Prompt
PROMPT = "Jack is looking at Anne. Anne is looking at George. Jack is married, George is not, and we don’t know if Anne is married. Is a married person looking at an unmarried person?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# System prompt
SYSTEM_PROMPT = "You are a logic bot designed to answer complex logic problems."

# Prompt
PROMPT = "Jack is looking at Anne. Anne is looking at George. Jack is married, George is not, and we don’t know if Anne is married. Is a married person looking at an unmarried person?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```
