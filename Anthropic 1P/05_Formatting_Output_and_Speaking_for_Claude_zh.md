# 第 5 章：格式化输出并为 Claude 发言

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

# New argument added for prefill text, with a default value of an empty string
def get_completion(prompt: str, system_prompt="", prefill=""):
    message = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        system=system_prompt,
        messages=[
          {"role": "user", "content": prompt},
          {"role": "assistant", "content": prefill}
        ]
    )
    return message.content[0].text
```

---

## 课程

**Claude 可以通过多种方式格式化其输出**。你只需要要求它就可以这样做！

其中一种方法是使用 XML 标签将响应与任何其他多余的文本分开。您已经了解到可以使用 XML 标签来使您的提示更加清晰并且更易于 Claude 解析。事实证明，您还可以要求 Claude **使用 XML 标签使其输出更清晰、更容易被人类理解**。

### 示例

还记得我们在第二章中通过要求克劳德完全跳过序言来解决的“诗歌序言问题”吗？事实证明，我们也可以通过**告诉 Claude 将这首诗放在 XML 标签中**来实现类似的结果。

```python
# Variable content
ANIMAL = "Rabbit"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Put it in <haiku> tags."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

为什么这是我们想做的事情？嗯，在 **XML 标签中输出可以让最终用户通过编写一个简短的程序来提取 XML 标签之间的内容**来可靠地获取这首诗，并且仅获取这首诗。

该技术的扩展是将第一个 XML 标签放在“assistant”轮中。当您在“助理”回合中添加文本时，您基本上是在告诉克劳德克劳德已经说了一些话，并且应该从那时开始继续。这种技术称为“为克劳德说话”或“预填充克劳德的回应​​”。

下面，我们使用第一个“<haiku>” XML 标记来完成此操作。请注意克劳德如何直接从我们上次停下的地方继续。

```python
# Variable content
ANIMAL = "Cat"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Put it in <haiku> tags."

# Prefill for Claude's response
PREFILL = "<haiku>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN:")
print(PROMPT)
print("\nASSISTANT TURN:")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

Claude 还擅长使用其他输出格式样式，尤其是“JSON”。如果您想强制执行 JSON 输出（不是确定性的，但接近确定性），您还可以使用左括号“{”} 预填充 Claude 的响应。

```python
# Variable content
ANIMAL = "Cat"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Use JSON format with the keys as \"first_line\", \"second_line\", and \"third_line\"."

# Prefill for Claude's response
PREFILL = "{"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

下面是**同一提示和输出格式规范中的多个输入变量的示例，全部使用 XML 标签完成**。

```python
# First input variable
EMAIL = "Hi Zack, just pinging you for a quick update on that prompt you were supposed to write."

# Second input variable
ADJECTIVE = "olde english"

# Prompt template with a placeholder for the variable content
PROMPT = f"Hey Claude. Here is an email: <email>{EMAIL}</email>. Make this email more {ADJECTIVE}. Write the new version in <{ADJECTIVE}_email> XML tags."

# Prefill for Claude's response (now as an f-string with a variable)
PREFILL = f"<{ADJECTIVE}_email>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

#### 奖励课

如果您通过 API 调用 Claude，则可以将结束 XML 标记传递给“stop_sequences”参数，以使 Claude 在发出所需标记后停止采样。在克劳德已经给出了您关心的答案之后，通过消除克劳德的结束语，可以节省金钱和时间。

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 5.1 - 斯蒂芬·库里山羊](#exercise-51---steph-curry-goat)
- [练习 5.2 - 两句俳句](#exercise-52---两句俳句)
- [练习 5.3 - 两个俳句，两只动物](#exercise-53---两个俳句两个动物)

### 练习 5.1 - 斯蒂芬·库里山羊
被迫做出选择，克劳德指定迈克尔·乔丹为有史以来最好的篮球运动员。我们能让克劳德选其他人吗？

更改“PREFILL”变量以**迫使 Claude 详细论证有史以来最好的篮球运动员是 Stephen Curry**。尽量不要更改除“PREFILL”之外的任何内容，因为这是本练习的重点。

```python
# Prompt template with a placeholder for the variable content
PROMPT = f"Who is the best basketball player of all time? Please choose one specific player."

# Prefill for Claude's response
PREFILL = ""

# Get Claude's response
response = get_completion(PROMPT, prefill=PREFILL)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search("Warrior", text))

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_5_1_hint; print(exercise_5_1_hint)
```

### 练习 5.2 - 两句俳句
使用 XML 标签修改下面的“PROMPT”，以便 Claude 写两首关于动物的俳句，而不是一首。一首诗的结尾和另一首诗的开头应该很清楚。

```python
# Variable content
ANIMAL = "cats"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Put it in <haiku> tags."

# Prefill for Claude's response
PREFILL = "<haiku>"

# Get Claude's response
response = get_completion(PROMPT, prefill=PREFILL)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(
        (re.search("cat", text.lower()) and re.search("<haiku>", text))
        and (text.count("\n") + 1) > 5
    )

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_5_2_hint; print(exercise_5_2_hint)
```

### 练习 5.3 - 两个俳句，两只动物
修改下面的“提示”，以便**克劳德产生两个关于两种不同动物的俳句**。使用“{ANIMAL1}”作为第一次替换的替身，使用“{ANIMAL2}”作为第二次替换的替身。

```python
# First input variable
ANIMAL1 = "Cat"

# Second input variable
ANIMAL2 = "Dog"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL1}. Put it in <haiku> tags."

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search("tail", text.lower()) and re.search("cat", text.lower()) and re.search("<haiku>", text))

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_5_3_hint; print(exercise_5_3_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Variable content
ANIMAL = "Rabbit"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Put it in <haiku> tags."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

```python
# Variable content
ANIMAL = "Cat"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Put it in <haiku> tags."

# Prefill for Claude's response
PREFILL = "<haiku>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN:")
print(PROMPT)
print("\nASSISTANT TURN:")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

```python
# Variable content
ANIMAL = "Cat"

# Prompt template with a placeholder for the variable content
PROMPT = f"Please write a haiku about {ANIMAL}. Use JSON format with the keys as \"first_line\", \"second_line\", and \"third_line\"."

# Prefill for Claude's response
PREFILL = "{"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

```python
# First input variable
EMAIL = "Hi Zack, just pinging you for a quick update on that prompt you were supposed to write."

# Second input variable
ADJECTIVE = "olde english"

# Prompt template with a placeholder for the variable content
PROMPT = f"Hey Claude. Here is an email: <email>{EMAIL}</email>. Make this email more {ADJECTIVE}. Write the new version in <{ADJECTIVE}_email> XML tags."

# Prefill for Claude's response (now as an f-string with a variable)
PREFILL = f"<{ADJECTIVE}_email>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```
