# 第 4 章：分离数据和指令

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

通常，我们不想编写完整的提示，而是希望**提示模板可以在提交给 Claude** 之前使用其他输入数据进行修改。如果您希望 Claude 每次都做同样的事情，但 Claude 用于其任务的数据每次可能不同，这可能会派上用场。

幸运的是，我们可以很容易地做到这一点，方法是**将提示的固定框架与可变的用户输入分开，然后将用户输入替换到提示中**，然后再将完整的提示发送给 Claude。

下面，我们将逐步介绍如何编写可替换的提示模板，以及如何替换用户输入。

### 示例

在第一个示例中，我们要求克劳德充当动物噪音发生器。请注意，提交给 Claude 的完整提示只是用输入（在本例中为“Cow”）替换的“PROMPT_TEMPLATE”。请注意，当我们打印出完整的提示时，单词“Cow”通过 f 字符串替换了“ANIMAL”占位符。

**注意：** 在实践中，您不必特别调用占位符变量。在这个例子中，我们将其称为“ANIMAL”，但同样简单，我们也可以将其称为“CREATURE”或“A”（尽管通常最好让变量名称具体且相关，这样即使没有替换，提示模板也很容易理解，只是为了用户可解析性）。只需确保您为变量命名的内容就是您用于提示模板 f 字符串的名称。

```python
# Variable content
ANIMAL = "Cow"

# Prompt template with a placeholder for the variable content
PROMPT = f"I will tell you the name of an animal. Please respond with the noise that animal makes. {ANIMAL}"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

为什么我们要像这样分离和替换输入？嗯，**提示模板简化了重复性任务**。假设您构建了一个提示结构，邀请第三方用户向提示提交内容（在本例中是他们想要生成声音的动物）。这些第三方用户不必编写甚至查看完整的提示。他们所要做的就是填写变量。

我们在这里使用变量和 f 字符串进行此替换，但您也可以使用 format() 方法来完成此替换。

**注意：** 提示模板可以包含任意多个变量！

当引入这样的替换变量时，**确保 Claude 知道变量在哪里开始和结束**（相对于指令或任务描述）非常重要。让我们看一个示例，其中指令和替换变量之间没有分隔。

对于我们人眼来说，下面的提示模板中变量的开始和结束位置是非常清楚的。然而，在完全替换的提示中，这种划分变得不清楚。

```python
# Variable content
EMAIL = "Show up at 6am tomorrow because I'm the CEO and I say so."

# Prompt template with a placeholder for the variable content
PROMPT = f"Yo Claude. {EMAIL} <----- Make this email more polite but don't change anything else about it."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

在这里，**Claude 认为“Yo Claude”是它应该重写的电子邮件的一部分**！你可以看出来，因为它以“亲爱的克劳德”开始重写。对于人眼来说，它很清晰，特别是在电子邮件开始和结束的提示模板中，但在替换后的提示中，它变得不太清晰。

我们如何解决这个问题？**将输入包装在 XML 标签中**！我们在下面执行了此操作，正如您所看到的，输出中不再有“Dear Claude”。

[XML 标签](https://docs.anthropic.com/claude/docs/use-xml-tags) 是尖括号标签，例如“<tag></tag>”。它们成对出现，由一个开始标记（例如“<tag>”）和一个由“/”标记的结束标记（例如“</tag>”）组成。XML 标签用于包裹内容，如下所示：`<tag>content</tag>`。

**注意：** 虽然 Claude 可以识别并使用各种分隔符和定界符，但我们建议您 **使用专门的 XML 标签作为 Claude 的分隔符**，因为 Claude 经过专门培训，可以识别 XML 标签作为提示组织机制。除了函数调用之外，**Claude 没有接受过任何特殊的 XML 标签培训，您应该使用它们来最大程度地提高性能**。我们特意通过这种方式让克劳德变得非常具有可塑性和可定制性。

```python
# Variable content
EMAIL = "Show up at 6am tomorrow because I'm the CEO and I say so."

# Prompt template with a placeholder for the variable content
PROMPT = f"Yo Claude. <email>{EMAIL}</email> <----- Make this email more polite but don't change anything else about it."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

让我们看看 XML 标签如何帮助我们的另一个例子。

在下面的提示中，**Claude 错误地解释了提示的哪一部分是指令，哪些部分是输入**。由于格式原因，它错误地将“每个都与一种动物有关，例如兔子”视为列表的一部分，而用户（填写“SENTENCES”变量的人）可能不希望这样。

```python
# Variable content
SENTENCES = """- I like how cows sound
- This sentence is about spiders
- This sentence may appear to be about dogs but it's actually about pigs"""

# Prompt template with a placeholder for the variable content
PROMPT = f"""Below is a list of sentences. Tell me the second item on the list.

- Each is about an animal, like rabbits.
{SENTENCES}"""

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

要解决这个问题，我们只需**将用户输入的句子放在 XML 标签中**。这向 Claude 显示了输入数据的开始和结束位置，尽管“每个数据都与动物有关，例如兔子”之前有误导性的连字符。

```python
# Variable content
SENTENCES = """- I like how cows sound
- This sentence is about spiders
- This sentence may appear to be about dogs but it's actually about pigs"""

# Prompt template with a placeholder for the variable content
PROMPT = f""" Below is a list of sentences. Tell me the second item on the list.

- Each is about an animal, like rabbits.
<sentences>
{SENTENCES}
</sentences>"""

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

**注意：** 在“每个都是关于动物”提示的错误版本中，我们必须包含连字符，以使克劳德按照我们在本示例中想要的方式做出错误响应。这是关于提示的重要一课：**小细节很重要**！**清除提示中的拼写错误和语法错误**总是值得的。Claude 对模式很敏感（在它的早期，在微调之前，它是一个原始的文本预测工具），当你犯错误时，它更有可能犯错误，当你听起来很聪明时，它会更聪明，当你听起来很愚蠢时，它会更愚蠢，等等。

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 4.1 - 俳句主题](#exercise-41---俳句主题)
- [练习 4.2 - 带有拼写错误的狗问题](#exercise-42---带有拼写错误的狗问题)
- [练习 4.3 - 狗问题第 2 部分](#exercise-42---dog-question-part-2)

### 练习 4.1 - 俳句主题
修改“PROMPT”，使其成为一个模板，该模板将接受名为“TOPIC”的变量并输出有关该主题的俳句。本练习只是为了测试您对使用 f 字符串的变量模板结构的理解。

```python
# Variable content
TOPIC = "Pigs"

# Prompt template with a placeholder for the variable content
PROMPT = f""

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search("pigs", text.lower()) and re.search("haiku", text.lower()))

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_4_1_hint; print(exercise_4_1_hint)
```

### 练习 4.2 - 带有拼写错误的狗问题
通过添加 XML 标签来修复“PROMPT”，以便 Claude 生成正确的答案。

尽量不要更改有关提示的任何其他内容。杂乱且错误百出的写作是故意的，所以你可以看到克劳德对这些错误的反应。

```python
# Variable content
QUESTION = "ar cn brown?"

# Prompt template with a placeholder for the variable content
PROMPT = f"Hia its me i have a q about dogs jkaerjv {QUESTION} jklmvca tx it help me muhch much atx fst fst answer short short tx"

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search("brown", text.lower()))

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_4_2_hint; print(exercise_4_2_hint)
```

### 练习 4.3 - 狗问题第 2 部分
修复“PROMPT”**不添加**添加 XML 标签。相反，只从提示中删除一两个单词。

正如上述练习一样，尽量不要更改提示的任何其他内容。这将向您展示克劳德可以解析和理解什么样的语言。

```python
# Variable content
QUESTION = "ar cn brown?"

# Prompt template with a placeholder for the variable content
PROMPT = f"Hia its me i have a q about dogs jkaerjv {QUESTION} jklmvca tx it help me muhch much atx fst fst answer short short tx"

# Get Claude's response
response = get_completion(PROMPT)

# Function to grade exercise correctness
def grade_exercise(text):
    return bool(re.search("brown", text.lower()))

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(response)
print("\n------------------------------------------ GRADING ------------------------------------------")
print("This exercise has been correctly solved:", grade_exercise(response))
```

❓ 如果您需要提示，请运行下面的单元格！

```python
from hints import exercise_4_3_hint; print(exercise_4_3_hint)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Variable content
ANIMAL = "Cow"

# Prompt template with a placeholder for the variable content
PROMPT = f"I will tell you the name of an animal. Please respond with the noise that animal makes. {ANIMAL}"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

```python
# Variable content
EMAIL = "Show up at 6am tomorrow because I'm the CEO and I say so."

# Prompt template with a placeholder for the variable content
PROMPT = f"Yo Claude. {EMAIL} <----- Make this email more polite but don't change anything else about it."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

```python
# Variable content
EMAIL = "Show up at 6am tomorrow because I'm the CEO and I say so."

# Prompt template with a placeholder for the variable content
PROMPT = f"Yo Claude. <email>{EMAIL}</email> <----- Make this email more polite but don't change anything else about it."

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

```python
# Variable content
SENTENCES = """- I like how cows sound
- This sentence is about spiders
- This sentence may appear to be about dogs but it's actually about pigs"""

# Prompt template with a placeholder for the variable content
PROMPT = f"""Below is a list of sentences. Tell me the second item on the list.

- Each is about an animal, like rabbits.
{SENTENCES}"""

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```

```python
# Variable content
SENTENCES = """- I like how cows sound
- This sentence is about spiders
- This sentence may appear to be about dogs but it's actually about pigs"""

# Prompt template with a placeholder for the variable content
PROMPT = f""" Below is a list of sentences. Tell me the second item on the list.

- Each is about an animal, like rabbits.
<sentences>
{SENTENCES}
</sentences>"""

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print(PROMPT)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT))
```
