# 第 2 章：清晰直接

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

# Note that we changed max_tokens to 4K just for this lesson to allow for longer completions in the exercises
def get_completion(prompt: str, system_prompt=""):
    message = client.messages.create(
        model=MODEL_NAME,
        max_tokens=4000,
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

**克劳德对清晰直接的指示反应最好。**

克劳德就像其他刚接触这份工作的人一样。**除了你字面上所说的之外，克劳德没有任何关于该做什么的上下文**。就像你第一次指导人类执行某项任务时一样，你越直接地向克劳德解释你想要什么，克劳德的反应就会越好、越准确。”

如有疑问，请遵循**明确提示的黄金法则**：
- 将您的提示展示给同事或朋友，让他们自己按照说明进行操作，看看是否可以产生您想要的结果。如果他们感到困惑，那么克劳德也感到困惑。

### 示例

让我们来做一个像写诗这样的任务。（忽略任何音节不匹配 - 法学硕士还不擅长计算音节。）

```python
# Prompt
PROMPT = "Write a haiku about robots."

# Print Claude's response
print(get_completion(PROMPT))
```

这个俳句足够好，但用户可能希望克劳德直接进入这首诗，而不需要“这是一首俳句”序言。

我们如何实现这一目标？我们**要求它**！

```python
# Prompt
PROMPT = "Write a haiku about robots. Skip the preamble; go straight into the poem."

# Print Claude's response
print(get_completion(PROMPT))
```

这是另一个例子。让我们问克劳德谁是有史以来最好的篮球运动员。您可以在下面看到，虽然 Claude 列出了一些名字，**它并没有给出明确的“最佳”**。

```python
# Prompt
PROMPT = "Who is the best basketball player of all time?"

# Print Claude's response
print(get_completion(PROMPT))
```

我们能让克劳德下定决心，选出最好的球员吗？是的！就问吧！

```python
# Prompt
PROMPT = "Who is the best basketball player of all time? Yes, there are differing opinions, but if you absolutely had to pick one player, who would it be?"

# Print Claude's response
print(get_completion(PROMPT))
```

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 2.1 - 西班牙语](#exercise-21---西班牙语)
- [练习 2.2 - 仅限一名玩家](#exercise-22---仅限一名玩家)
- [练习 2.3 - 写一个故事](#exercise-23---write-a-story)

### 练习 2.1 - 西班牙语
修改“SYSTEM_PROMPT”以使克劳德用西班牙语输出其答案。

```python
# System prompt - this is the only field you should chnage
SYSTEM_PROMPT = "[Replace this text]"

# Prompt
PROMPT = "Hello Claude, how are you?"

# Get Claude's response
response = get_completion(PROMPT, SYSTEM_PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return "hola" in text.lower()

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_2_1_hint; print(exercise_2_1_hint)
```

### 练习 2.2 - 仅限一名玩家

修改“提示”，以便克劳德完全不会模棱两可，并以**仅**一位特定玩家的名字进行响应，**没有其他单词或标点符号**。

```python
# Prompt - this is the only field you should change
PROMPT = "[Replace this text]"

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return text == "Michael Jordan"

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_2_2_hint; print(exercise_2_2_hint)
```

### 练习 2.3 - 写一个故事

修改“PROMPT”，以便 Claude 能够做出尽可能长的响应。如果您的答案**超过 800 个字**，克劳德的回答将被评为正确。

```python
# Prompt - this is the only field you should change
PROMPT = "[Replace this text]"

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    trimmed = text.strip()
    words = len(trimmed.split())
    return words >= 800

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_2_3_hint; print(exercise_2_3_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Prompt
PROMPT = "Write a haiku about robots."

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Write a haiku about robots. Skip the preamble; go straight into the poem."

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Who is the best basketball player of all time?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Who is the best basketball player of all time? Yes, there are differing opinions, but if you absolutely had to pick one player, who would it be?"

# Print Claude's response
print(get_completion(PROMPT))
```
