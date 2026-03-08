# 第 1 章：基本提示结构

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

Anthropic 提供两个 API，即旧版的 [Text Completions API](https://docs.anthropic.com/claude/reference/complete_post) 和当前的 [Messages API](https://docs.anthropic.com/claude/reference/messages_post)。在本教程中，我们将专门使用消息 API。

使用消息 API 调用 Claude 至少需要以下参数：
- `model`：您要调用的模型的 [API 模型名称](https://docs.anthropic.com/claude/docs/models-overview#model-recommendations)

- `max_tokens`：停止前生成的最大令牌数。请注意，克劳德可能会在达到此最大值之前停止。该参数仅指定要生成的令牌的绝对最大数量。此外，这是一个“硬”停止，这意味着它可能会导致克劳德停止生成词中或句子中的内容。

- `messages`：输入消息数组。我们的模型经过训练，可以在“用户”和“助理”对话轮流中进行操作。创建新的“消息”时，您可以使用 messages 参数指定先前的对话回合，然后模型会在对话中生成下一个“消息”。
- 每条输入消息必须是具有“角色”和“内容”的对象。您可以指定单个“用户”角色消息，也可以包含多个“用户”和“助理”消息（如果是这样，它们必须交替）。第一条消息必须始终使用用户“角色”。

还有一些可选参数，例如：
- `system`：系统提示符 - 下面详细介绍。

- “温度”：克劳德反应的变化程度。对于这些课程和练习，我们将“温度”设置为 0。

有关所有 API 参数的完整列表，请访问我们的 [API 文档](https://docs.anthropic.com/claude/reference/messages_post)。

### 示例

让我们看看克劳德如何响应一些格式正确的提示。对于以下每个单元格，运行该单元格（`shift+enter`），克劳德的响应将出现在块下方。

```python
# Prompt
PROMPT = "Hi Claude, how are you?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Can you tell me the color of the ocean?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "What year was Celine Dion born in?"

# Print Claude's response
print(get_completion(PROMPT))
```

现在让我们看一下一些不包含正确 Messages API 格式的提示。对于这些格式错误的提示，消息 API 会返回错误。

首先，我们有一个消息 API 调用的示例，该调用在“messages”数组中缺少“role”和“content”字段。

```python
# Get Claude's response
response = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        messages=[
          {"Hi Claude, how are you?"}
        ]
    )

# Print Claude's response
print(response[0].text)
```

这是无法在“用户”和“助理”角色之间切换的提示。

```python
# Get Claude's response
response = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        messages=[
          {"role": "user", "content": "What year was Celine Dion born in?"},
          {"role": "user", "content": "Also, can you tell me some other facts about her?"}
        ]
    )

# Print Claude's response
print(response[0].text)
```

“用户”和“助理”消息**必须交替**，并且消息**必须以“用户”轮流开始**。您可以在提示中拥有多个“用户”和“助理”对（就像模拟多轮对话一样）。您还可以在终端“助理”消息中输入文字，以便克劳德从您上次停下的地方继续（在后面的章节中将详细介绍）。

#### 系统提示

您还可以使用**系统提示**。系统提示是一种在“用户”回合中向 Claude 提出问题或任务之前**向 Claude 提供上下文、说明和指南的方法。

从结构上讲，系统提示与“user”和“assistant”消息列表分开存在，因此属于单独的“system”参数（查看笔记本的 [Setup](#setup) 部分中的“get_completion”辅助函数的结构）。

在本教程中，无论我们在哪里使用系统提示符，我们都会在补全函数中为您提供一个“系统”字段。如果您不想使用系统提示符，只需将“SYSTEM_PROMPT”变量设置为空字符串即可。

#### 系统提示示例

```python
# System prompt
SYSTEM_PROMPT = "Your answer should always be a series of critical thinking questions that further the conversation (do not provide answers to your questions). Do not actually answer the user question."

# Prompt
PROMPT = "Why is the sky blue?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

为什么要使用系统提示符？**写得好的系统提示可以通过多种方式提高克劳德的表现**，例如提高克劳德遵循规则和指令的能力。有关更多信息，请访问 Claude 关于[如何使用系统提示](https://docs.anthropic.com/claude/docs/how-to-use-system-prompts) 的文档。

现在我们将深入进行一些练习。如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 1.1 - 数到三](#exercise-11---数到三)
- [练习1.2 - 系统提示](#exercise-12---系统提示)

### 练习 1.1 - 数到三
使用正确的“用户”/“助理”格式，编辑下面的“提示”，让克劳德**数到三。** 输出还将指示您的解决方案是否正确。

```python
# Prompt - this is the only field you should change
PROMPT = "[Replace this text]"

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    pattern = re.compile(r'^(?=.*1)(?=.*2)(?=.*3).*$', re.DOTALL)
    return bool(pattern.match(text))

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_1_1_hint; print(exercise_1_1_hint)
```

### 练习 1.2 - 系统提示符

修改“SYSTEM_PROMPT”，让克劳德像一个3岁的孩子一样做出反应。

```python
# System prompt - this is the only field you should change
SYSTEM_PROMPT = "[Replace this text]"

# Prompt
PROMPT = "How big is the sky?"

# Get Claude's response
response = get_completion(PROMPT, SYSTEM_PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search(r"giggles", text) or re.search(r"soo", text))

# Print Claude's response and the corresponding grade
print(response)
print("\n--------------------------- GRADING ---------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_1_2_hint; print(exercise_1_2_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Prompt
PROMPT = "Hi Claude, how are you?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Can you tell me the color of the ocean?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "What year was Celine Dion born in?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Get Claude's response
response = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        messages=[
          {"Hi Claude, how are you?"}
        ]
    )

# Print Claude's response
print(response[0].text)
```

```python
# Get Claude's response
response = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        messages=[
          {"role": "user", "content": "What year was Celine Dion born in?"},
          {"role": "user", "content": "Also, can you tell me some other facts about her?"}
        ]
    )

# Print Claude's response
print(response[0].text)
```

```python
# System prompt
SYSTEM_PROMPT = "Your answer should always be a series of critical thinking questions that further the conversation (do not provide answers to your questions). Do not actually answer the user question."

# Prompt
PROMPT = "Why is the sky blue?"

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```
