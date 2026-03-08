# 第 7 章：使用示例（少量提示）

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

**给 Claude 举例说明你希望它如何表现（或你不希望它如何表现）对于以下方面非常有效**：
- 得到正确的答案
- 以正确的格式获得答案

这种提示也称为“**少镜头提示**”。您可能还会遇到“零次”、“n次”或“一次”这样的短语。“镜头”的数量是指提示中使用了多少示例。

### 示例

假设您是一名开发人员，正在尝试构建一个“家长机器人”来回答孩子们的问题。**克劳德的默认反应非常正式和机械**。这会伤透孩子的心。

```python
# Prompt
PROMPT = "Will Santa bring me presents on Christmas?"

# Print Claude's response
print(get_completion(PROMPT))
```

您可以花时间描述您想要的语气，但**给克劳德提供一些理想回应的例子**会容易得多。

```python
# Prompt
PROMPT = """Please complete the conversation by writing the next line, speaking as "A".
Q: Is the tooth fairy real?
A: Of course, sweetie. Wrap up your tooth and put it under your pillow tonight. There might be something waiting for you in the morning.
Q: Will Santa bring me presents on Christmas?"""

# Print Claude's response
print(get_completion(PROMPT))
```

在下面的格式化示例中，我们可以逐步引导 Claude 完成一组有关如何提取姓名和职业的格式化说明，然后完全按照我们想要的方式对其进行格式化，或者我们可以**为 Claude 提供一些格式正确的示例，然后 Claude 可以从中推断**。注意“助理”回合中的“<个人>”，让克劳德从右脚开始。

```python
# Prompt template with a placeholder for the variable content
PROMPT = """Silvermist Hollow, a charming village, was home to an extraordinary group of individuals.
Among them was Dr. Liam Patel, a neurosurgeon who revolutionized surgical techniques at the regional medical center.
Olivia Chen was an innovative architect who transformed the village's landscape with her sustainable and breathtaking designs.
The local theater was graced by the enchanting symphonies of Ethan Kovacs, a professionally-trained musician and composer.
Isabella Torres, a self-taught chef with a passion for locally sourced ingredients, created a culinary sensation with her farm-to-table restaurant, which became a must-visit destination for food lovers.
These remarkable individuals, each with their distinct talents, contributed to the vibrant tapestry of life in Silvermist Hollow.
<individuals>
1. Dr. Liam Patel [NEUROSURGEON]
2. Olivia Chen [ARCHITECT]
3. Ethan Kovacs [MISICIAN AND COMPOSER]
4. Isabella Torres [CHEF]
</individuals>

At the heart of the town, Chef Oliver Hamilton has transformed the culinary scene with his farm-to-table restaurant, Green Plate. Oliver's dedication to sourcing local, organic ingredients has earned the establishment rave reviews from food critics and locals alike.
Just down the street, you'll find the Riverside Grove Library, where head librarian Elizabeth Chen has worked diligently to create a welcoming and inclusive space for all. Her efforts to expand the library's offerings and establish reading programs for children have had a significant impact on the town's literacy rates.
As you stroll through the charming town square, you'll be captivated by the beautiful murals adorning the walls. These masterpieces are the work of renowned artist, Isabella Torres, whose talent for capturing the essence of Riverside Grove has brought the town to life.
Riverside Grove's athletic achievements are also worth noting, thanks to former Olympic swimmer-turned-coach, Marcus Jenkins. Marcus has used his experience and passion to train the town's youth, leading the Riverside Grove Swim Team to several regional championships.
<individuals>
1. Oliver Hamilton [CHEF]
2. Elizabeth Chen [LIBRARIAN]
3. Isabella Torres [ARTIST]
4. Marcus Jenkins [COACH]
</individuals>

Oak Valley, a charming small town, is home to a remarkable trio of individuals whose skills and dedication have left a lasting impact on the community.
At the town's bustling farmer's market, you'll find Laura Simmons, a passionate organic farmer known for her delicious and sustainably grown produce. Her dedication to promoting healthy eating has inspired the town to embrace a more eco-conscious lifestyle.
In Oak Valley's community center, Kevin Alvarez, a skilled dance instructor, has brought the joy of movement to people of all ages. His inclusive dance classes have fostered a sense of unity and self-expression among residents, enriching the local arts scene.
Lastly, Rachel O'Connor, a tireless volunteer, dedicates her time to various charitable initiatives. Her commitment to improving the lives of others has been instrumental in creating a strong sense of community within Oak Valley.
Through their unique talents and unwavering dedication, Laura, Kevin, and Rachel have woven themselves into the fabric of Oak Valley, helping to create a vibrant and thriving small town."""

# Prefill for Claude's response
PREFILL = "<individuals>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN:")
print(PROMPT)
print("\nASSISTANT TURN:")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 7.1 - 通过示例设置电子邮件格式](#exercise-71---email-formatting-via-examples)

### 练习 7.1 - 通过示例设置电子邮件格式
我们将重做练习 6.2，但这一次，我们将编辑“提示”以使用电子邮件的“少量”示例 + 正确的分类（和格式），以使 Claude 输出正确的答案。我们希望 Claude 输出的*最后一个*字母是类别的字母。

如果您忘记每封电子邮件的正确字母类别，请参阅“电子邮件”列表中每封电子邮件旁边的注释。

请记住，这些是电子邮件的类别：
- (A) 售前问题
- (B) 破损或有缺陷的物品
- (C) 计费问题
- (D) 其他（请解释）

```python
# Prompt template with a placeholder for the variable content
PROMPT = """Please classify this email as either green or blue: {email}"""

# Prefill for Claude's response
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

# Iterate through list of emails
for i,email in enumerate(EMAILS):

    # Substitute the email text into the email placeholder variable
    formatted_prompt = PROMPT.format(email=email)

    # Get Claude's response
    response = get_completion(formatted_prompt, prefill=PREFILL)

    # Grade Claude's response
    grade = any([bool(re.search(ans, response[-1])) for ans in ANSWERS[i]])

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
from hints import exercise_7_1_hint; print(exercise_7_1_hint)
```

还卡住了吗？运行下面的单元格以获取示例解决方案。

```python
from hints import exercise_7_1_solution; print(exercise_7_1_solution)
```

### 恭喜！

如果您已经完成了到目前为止的所有练习，那么您就可以进入下一章了。快乐提示！

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
# Prompt
PROMPT = "Will Santa bring me presents on Christmas?"

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt
PROMPT = """Please complete the conversation by writing the next line, speaking as "A".
Q: Is the tooth fairy real?
A: Of course, sweetie. Wrap up your tooth and put it under your pillow tonight. There might be something waiting for you in the morning.
Q: Will Santa bring me presents on Christmas?"""

# Print Claude's response
print(get_completion(PROMPT))
```

```python
# Prompt template with a placeholder for the variable content
PROMPT = """Silvermist Hollow, a charming village, was home to an extraordinary group of individuals.
Among them was Dr. Liam Patel, a neurosurgeon who revolutionized surgical techniques at the regional medical center.
Olivia Chen was an innovative architect who transformed the village's landscape with her sustainable and breathtaking designs.
The local theater was graced by the enchanting symphonies of Ethan Kovacs, a professionally-trained musician and composer.
Isabella Torres, a self-taught chef with a passion for locally sourced ingredients, created a culinary sensation with her farm-to-table restaurant, which became a must-visit destination for food lovers.
These remarkable individuals, each with their distinct talents, contributed to the vibrant tapestry of life in Silvermist Hollow.
<individuals>
1. Dr. Liam Patel [NEUROSURGEON]
2. Olivia Chen [ARCHITECT]
3. Ethan Kovacs [MISICIAN AND COMPOSER]
4. Isabella Torres [CHEF]
</individuals>

At the heart of the town, Chef Oliver Hamilton has transformed the culinary scene with his farm-to-table restaurant, Green Plate. Oliver's dedication to sourcing local, organic ingredients has earned the establishment rave reviews from food critics and locals alike.
Just down the street, you'll find the Riverside Grove Library, where head librarian Elizabeth Chen has worked diligently to create a welcoming and inclusive space for all. Her efforts to expand the library's offerings and establish reading programs for children have had a significant impact on the town's literacy rates.
As you stroll through the charming town square, you'll be captivated by the beautiful murals adorning the walls. These masterpieces are the work of renowned artist, Isabella Torres, whose talent for capturing the essence of Riverside Grove has brought the town to life.
Riverside Grove's athletic achievements are also worth noting, thanks to former Olympic swimmer-turned-coach, Marcus Jenkins. Marcus has used his experience and passion to train the town's youth, leading the Riverside Grove Swim Team to several regional championships.
<individuals>
1. Oliver Hamilton [CHEF]
2. Elizabeth Chen [LIBRARIAN]
3. Isabella Torres [ARTIST]
4. Marcus Jenkins [COACH]
</individuals>

Oak Valley, a charming small town, is home to a remarkable trio of individuals whose skills and dedication have left a lasting impact on the community.
At the town's bustling farmer's market, you'll find Laura Simmons, a passionate organic farmer known for her delicious and sustainably grown produce. Her dedication to promoting healthy eating has inspired the town to embrace a more eco-conscious lifestyle.
In Oak Valley's community center, Kevin Alvarez, a skilled dance instructor, has brought the joy of movement to people of all ages. His inclusive dance classes have fostered a sense of unity and self-expression among residents, enriching the local arts scene.
Lastly, Rachel O'Connor, a tireless volunteer, dedicates her time to various charitable initiatives. Her commitment to improving the lives of others has been instrumental in creating a strong sense of community within Oak Valley.
Through their unique talents and unwavering dedication, Laura, Kevin, and Rachel have woven themselves into the fabric of Oak Valley, helping to create a vibrant and thriving small town."""

# Prefill for Claude's response
PREFILL = "<individuals>"

# Print Claude's response
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN:")
print(PROMPT)
print("\nASSISTANT TURN:")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```
