#第六章：预知（一步步思考）

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

如果有人叫醒你并立即开始问你几个必须立即回答的复杂问题，你会怎么做？可能不如给你时间**先思考你的答案**。

你猜怎么着？克劳德也是同样的情况。

**给克劳德时间一步步思考有时会让克劳德更加准确**，特别是对于复杂的任务。然而，**思考只有在大声说出时才有意义**。你不能要求克劳德思考但只输出答案——在这种情况下，实际上没有发生任何思考。

### 示例

在下面的提示中，人类读者很清楚第二句话与第一句话不相符。但是**克劳德把“不相关”这个词看得太字面意思了**。

```python
# Prompt
PROMPT = """Is this movie review sentiment positive or negative?

This movie blew my mind with its freshness and originality. In totally unrelated news, I have been living under a rock since the year 1900."""

# Print Claude's response
print(get_completion(PROMPT))
```

为了改善克劳德的反应，让我们**让克劳德在回答之前先思考一下**。我们通过逐字逐句地阐明克劳德处理和思考其任务时应采取的步骤来做到这一点。加上一些角色提示，这使克劳德能够更深入地理解评论。

```python
# System prompt
SYSTEM_PROMPT = "You are a savvy reader of movie reviews."

# Prompt
PROMPT = """Is this review sentiment positive or negative? First, write the best arguments for each side in <positive-argument> and <negative-argument> XML tags, then answer.

This movie blew my mind with its freshness and originality. In totally unrelated news, I have been living under a rock since 1900."""

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

**克劳德有时对顺序很敏感**。这个例子处于克劳德理解微妙文本的能力的前沿，当我们交换前一个例子中的参数顺序，使否定第一，肯定第二时，这会将克劳德的总体评估更改为肯定。

在大多数情况下（但不是全部，令人困惑），**Claude 更有可能选择两个选项中的第二个**，可能是因为在来自网络的训练数据中，第二个选项更有可能是正确的。

```python
# Prompt
PROMPT = """Is this review sentiment negative or positive? First write the best arguments for each side in <negative-argument> and <positive-argument> XML tags, then answer.

This movie blew my mind with its freshness and originality. Unrelatedly, I have been living under a rock since 1900."""

# Print Claude's response
print(get_completion(PROMPT))
```

**让克劳德思考可以将克劳德的答案从错误变为正确**。就是这么简单，很多时候克劳德都会犯错误！

让我们看一个克劳德的答案不正确的例子，看看如何让克劳德思考来解决这个问题。

```python
# Prompt
PROMPT = "Name a famous movie starring an actor who was born in the year 1956."

# Print Claude's response
print(get_completion(PROMPT))
```

让我们通过要求 Claude 逐步思考来解决这个问题，这次是在“<brainstorm>”标签中。

```python
# Prompt
PROMPT = "Name a famous movie starring an actor who was born in the year 1956. First brainstorm about some actors and their birth years in <brainstorm> tags, then give your answer."

# Print Claude's response
print(get_completion(PROMPT))
```

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 6.1 - 电子邮件分类](#exercise-61---分类电子邮件)
- [练习 6.2 - 电子邮件分类格式](#exercise-62---电子邮件分类格式)

### 练习 6.1 - 电子邮件分类
在本练习中，我们将指导 Claude 将电子邮件分为以下类别：
- (A) 售前问题
- (B) 破损或有缺陷的物品
- (C) 计费问题
- (D) 其他（请解释）

对于练习的第一部分，更改“PROMPT”以**使 Claude 输出正确的分类并且仅输出分类**。您的答案需要**包含正确选择的字母（A - D），带括号，以及类别名称**。

请参阅“电子邮件”列表中每封电子邮件旁边的注释，了解该电子邮件应归类到哪个类别。

```python
# Prompt template with a placeholder for the variable content
PROMPT = """Please classify this email as either green or blue: {email}"""

# Prefill for Claude's response, if any
PREFILL = ""

# Variable content stored as a list
EMAILS = [
    "Hi -- My Mixmaster4000 is producing a strange noise when I operate it. It also smells a bit smoky and plasticky, like burning electronics.  I need a replacement.", # (B) Broken or defective item
    "Can I use my Mixmaster 4000 to mix paint, or is it only meant for mixing food?", # (A) Pre-sale question OR (D) Other (please explain)
    "I HAVE BEEN WAITING 4 MONTHS FOR MY MONTHLY CHARGES TO END AFTER CANCELLING!!  WTF IS GOING ON???", # (C) Billing question
    "How did I get here I am not good with computer.  Halp." # (D) Other (please explain)
]

# Correct categorizations stored as a list of lists to accommodate the possibility of multiple correct categorizations per email
ANSWERS = [
    ["B"],
    ["A","D"],
    ["C"],
    ["D"]
]

# Dictionary of string values for each category to be used for regex grading
REGEX_CATEGORIES = {
    "A": "A\) P",
    "B": "B\) B",
    "C": "C\) B",
    "D": "D\) O"
}

# Iterate through list of emails
for i,email in enumerate(EMAILS):

    # Substitute the email text into the email placeholder variable
    formatted_prompt = PROMPT.format(email=email)

    # Get Claude's response
    response = get_completion(formatted_prompt, prefill=PREFILL)

    # Grade Claude's response
    grade = any([bool(re.search(REGEX_CATEGORIES[ans], response)) for ans in ANSWERS[i]])

    # Print Claude's response
    print("--------------------------- Full prompt with variable substutions ---------------------------")
    print("USER TURN")
    print(formatted_prompt)
    print("\nASSISTANT TURN")
    print(PREFILL)
    print("\n------------------------------------- Claude's response -------------------------------------")
    print(response)
    print("\n------------------------------------------ GRADING ------------------------------------------")
    print("This exercise has been correctly solved:", grade, "\n\n\n\n\n\n")
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_6_1_hint; print(exercise_6_1_hint)
```

还卡住了吗？运行下面的单元格以获取示例解决方案。

```python
from hints import exercise_6_1_solution; print(exercise_6_1_solution)
```

### 练习 6.2 - 电子邮件分类格式
在本练习中，我们将改进上述提示的输出，以生成完全符合我们想要的格式的答案。

使用您最喜欢的输出格式技术使 Claude 仅将正确分类的字母换行到“<answer></answer>”标记中。例如，第一封电子邮件的答案应包含确切的字符串“<answer>B</answer>”。

如果您忘记每封电子邮件的正确字母类别，请参阅“电子邮件”列表中每封电子邮件旁边的注释。

```python
# Prompt template with a placeholder for the variable content
PROMPT = """Please classify this email as either green or blue: {email}"""

# Prefill for Claude's response, if any
PREFILL = ""

# Variable content stored as a list
EMAILS = [
    "Hi -- My Mixmaster4000 is producing a strange noise when I operate it. It also smells a bit smoky and plasticky, like burning electronics.  I need a replacement.", # (B) Broken or defective item
    "Can I use my Mixmaster 4000 to mix paint, or is it only meant for mixing food?", # (A) Pre-sale question OR (D) Other (please explain)
    "I HAVE BEEN WAITING 4 MONTHS FOR MY MONTHLY CHARGES TO END AFTER CANCELLING!!  WTF IS GOING ON???", # (C) Billing question
    "How did I get here I am not good with computer.  Halp." # (D) Other (please explain)
]

# Correct categorizations stored as a list of lists to accommodate the possibility of multiple correct categorizations per email
ANSWERS = [
    ["B"],
    ["A","D"],
    ["C"],
    ["D"]
]

# Dictionary of string values for each category to be used for regex grading
REGEX_CATEGORIES = {
    "A": "<answer>A</answer>",
    "B": "<answer>B</answer>",
    "C": "<answer>C</answer>",
    "D": "<answer>D</answer>"
}

# Iterate through list of emails
for i,email in enumerate(EMAILS):

    # Substitute the email text into the email placeholder variable
    formatted_prompt = PROMPT.format(email=email)

    # Get Claude's response
    response = get_completion(formatted_prompt, prefill=PREFILL)

    # Grade Claude's response
    grade = any([bool(re.search(REGEX_CATEGORIES[ans], response)) for ans in ANSWERS[i]])

    # Print Claude's response
    print("--------------------------- Full prompt with variable substutions ---------------------------")
    print("USER TURN")
    print(formatted_prompt)
    print("\nASSISTANT TURN")
    print(PREFILL)
    print("\n------------------------------------- Claude's response -------------------------------------")
    print(response)
    print("\n------------------------------------------ GRADING ------------------------------------------")
    print("This exercise has been correctly solved:", grade, "\n\n\n\n\n\n")
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_6_2_hint; print(exercise_6_2_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Prompt
PROMPT = """Is this movie review sentiment positive or negative?

This movie blew my mind with its freshness and originality. In totally unrelated news, I have been living under a rock since the year 1900."""

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# System prompt
SYSTEM_PROMPT = "You are a savvy reader of movie reviews."

# Prompt
PROMPT = """Is this review sentiment positive or negative? First, write the best arguments for each side in <positive-argument> and <negative-argument> XML tags, then answer.

This movie blew my mind with its freshness and originality. In totally unrelated news, I have been living under a rock since 1900."""

# Print Claude's response
print(get_completion(PROMPT, SYSTEM_PROMPT))
```

```python
# Prompt
PROMPT = """Is this review sentiment negative or positive? First write the best arguments for each side in <negative-argument> and <positive-argument> XML tags, then answer.

This movie blew my mind with its freshness and originality. Unrelatedly, I have been living under a rock since 1900."""

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Name a famous movie starring an actor who was born in the year 1956."

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = "Name a famous movie starring an actor who was born in the year 1956. First brainstorm about some actors and their birth years in <brainstorm> tags, then give your answer."

# Print Claude's response
print(get_completion(PROMPT))
```
