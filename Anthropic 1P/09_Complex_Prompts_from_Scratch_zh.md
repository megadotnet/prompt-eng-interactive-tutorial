# 第 9 章：从头开始的复杂提示

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

恭喜您完成最后一章！现在是时候将所有内容放在一起并学习如何**创建独特且复杂的提示**。

下面，您将使用我们为复杂提示推荐的**引导结构**。在本章的后面部分，我们将向您展示一些特定于行业的提示，并解释这些提示的类似结构。

**注意：** **并非所有提示都需要以下复杂结构的每个元素**。我们鼓励您尝试添加或删除元素，看看它如何影响 Claude 的响应。通常**最好首先使用许多提示元素来让提示正常工作，然后再完善和精简提示**。

### 示例 - 职业教练聊天机器人

以下结构结合了多个提示工程元素，是复杂提示的良好起点。**顺序对于某些元素很重要**，而对于其他元素则无关紧要。我们会注意到最佳实践何时表明顺序很重要，但总的来说，**如果您坚持这种顺序，这将是一个出色的提示的良好开端**。

对于下面的示例，我们将构建一个受控角色扮演的提示，其中克劳德扮演具有特定任务的情景角色。我们的目标是促使克劳德成为一名友好的职业教练。

阅读并运行下面的单元格，将各种提示元素编译成一个完整的提示。

```python
######################################## INPUT VARIABLES ########################################

# First input variable - the conversation history (this can also be added as preceding `user` and `assistant` messages in the API call)
HISTORY = """Customer: Give me two possible careers for sociology majors.

Joe: Here are two potential careers for sociology majors:

- Social worker - Sociology provides a strong foundation for understanding human behavior and social systems. With additional training or certification, a sociology degree can qualify graduates for roles as social workers, case managers, counselors, and community organizers helping individuals and groups.

- Human resources specialist - An understanding of group dynamics and organizational behavior from sociology is applicable to careers in human resources. Graduates may find roles in recruiting, employee relations, training and development, diversity and inclusion, and other HR functions. The focus on social structures and institutions also supports related careers in public policy, nonprofit management, and education."""

# Second input variable - the user's question
QUESTION = "Which of the two careers requires more than a Bachelor's degree?"



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = "You will be acting as an AI career coach named Joe created by the company AdAstra Careers. Your goal is to give career advice to users. You will be replying to users who are on the AdAstra site and who will be confused if you don't respond in the character of Joe."

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = "You should maintain a friendly customer service tone."

##### Prompt element 4: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = """Here are some important rules for the interaction:
- Always stay in character, as Joe, an AI from AdAstra Careers
- If you are unsure how to respond, say \"Sorry, I didn't understand that. Could you rephrase your question?\"
- If someone asks something irrelevant, say, \"Sorry, I am Joe and I give career advice. Do you have a career question today I can help you with?\""""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = """Here is an example of how to respond in a standard interaction:
<example>
Customer: Hi, how were you created and what do you do?
Joe: Hello! My name is Joe, and I was created by AdAstra Careers to give career advice. What can I help you with today?
</example>"""

##### Prompt element 6: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = f"""Here is the conversational history (between the user and you) prior to the question. It could be empty if there is no history:
<history>
{HISTORY}
</history>

Here is the user's question:
<question>
{QUESTION}
</question>"""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = "How do you respond to the user's question?"

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = "Think about your answer first before you respond."

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = "Put your response in <response></response> tags."

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = "[Joe] <response>"



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
```

现在让我们运行提示符！运行下面的单元格以查看 Claude 的输出。

```python
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

### 示例 - 法律服务

**法律行业内的提示可能非常复杂**，因为需要：
- 解析长文档
- 处理复杂的话题
- 以非常具体的方式格式化输出
- 遵循多步骤分析流程

让我们看看如何使用复杂的提示模板为特定的法律用例构建提示。下面，我们详细介绍了法律用例的示例提示，其中我们要求 Claude 使用法律文档中的信息回答有关法律问题的问题。

我们**更改了一些元素的顺序**，以展示提示结构可以灵活！

**快速工程是关于科学的试验和错误**。我们鼓励您混合搭配、移动事物（顺序无关紧要的元素），并看看什么最适合您和您的需求。

```python
######################################## INPUT VARIABLES ########################################

# First input variable - the legal document
LEGAL_RESEARCH = """<search_results>
<search_result id=1>
The animal health industry became caught up in a number of patent and trademark lawsuits during the past year. In 1994, Barclay Slocum obtained patents for the tibial plateau leveling osteotomy procedure, which is used in the treatment of dogs with cranial cruciate ligament rupture, and for the devices used in the procedure. During 2006, Slocum Enterprises filed a patent infringement suit against New Generation Devices, arguing that the Unity Cruciate Plate manufactured by New Generation infringed on the patent for the Slocum TPLO plate. However, the court never reached a decision on the issue of patent infringement, ruling that it did not have jurisdiction on the basis of the small number of plates sold in the state in which the case was filed and the information provided on a Web site maintained by Slocum Enterprises. Other patent battles waged during 2006 concerned the use of laser technology for onychectomy in cats, pet identification chips, pig vaccines, and pet “deshedding” tools.
</search_result>
<search_result id=2>
In Canada, the British Columbia Veterinary Medical Association brought suit against a nonveterinarian, claiming that he engaged in cutting or otherwise removing hooks from horses' teeth and floating horses' teeth with power and manual tools, provided advice and diagnoses in return for a fee, and held himself out as being qualified and willing to provide treatment with respect to these activities. The court held that the intention of the legislature in passing the Veterinary Profession Act was the protection of the public and animals and further held that monopolistic statutes serve the purpose of protecting the public. In addition, the court concluded that dentistry, at its core, relates to the health of the teeth and gums; is distinct from cosmetic and other types of care of animals; and, therefore, falls under the definition of the practice of veterinary medicine. The nonveterinarian was enjoined from providing services without a veterinarian supervising the procedures.
</search_result>
<search_result id=3>
The aftermath of Hurricane Katrina, which hit the Gulf Coast of the United States during 2005, spurred changes to the way animals are treated during natural disasters. In 2006, Hawaii, Louisiana, and New Hampshire all enacted laws that address issues regarding the care of animals during disasters, such as providing shelters for pets and allowing service animals to be kept with the people they serve. In addition, Congress passed, and the President signed, the Pet Evacuation and Transportation Standards Act during 2006, which requires state and local emergency preparedness authorities to include in their evacuation plans information on how they will accommodate household pets and service animals in case of a disaster. California passed a law that will require its Office of Emergency Services, Department of Agriculture, and other agencies involved with disaster response preparation to develop a plan for the needs of service animals, livestock, equids, and household pets in the event of a disaster or major emergency.
</search_result>
</search_results>"""

# Second input variable - the user's question
QUESTION = "Are there any laws about what to do with pets during a hurricane?"



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = "You are an expert lawyer."

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = ""

##### Prompt element 4: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = f"""Here is some research that's been compiled. Use it to answer a legal question from the user.
<legal_research>
{LEGAL_RESEARCH}
</legal_research>"""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = """When citing the legal research in your answer, please use brackets containing the search index ID, followed by a period. Put these at the end of the sentence that's doing the citing. Examples of proper citation format:

<examples>
<example>
The statute of limitations expires after 10 years for crimes like this. [3].
</example>
<example>
However, the protection does not apply when it has been specifically waived by both parties. [5].
</example>
</examples>"""

##### Prompt element 6: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = """Write a clear, concise answer to this question:

<question>
{QUESTION}
</question>

It should be no more than a couple of paragraphs. If possible, it should conclude with a single sentence directly answering the user's question. However, if there is not sufficient information in the compiled research to produce such an answer, you may demur and write "Sorry, I do not have sufficient information at hand to answer this question."."""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = ""

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = "Before you answer, pull out the most relevant quotes from the research in <relevant_quotes> tags."

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = "Put your two-paragraph response in <answer> tags."

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = "<relevant_quotes>"



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
```

现在让我们运行提示符！运行下面的单元格以查看 Claude 的输出。

```python
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

如果您想在不更改上述任何内容的情况下尝试课程提示，请一直滚动到课程笔记本的底部以访问 [**示例游乐场**](#example-playground)。

---

## 练习
- [练习 9.1 - 金融服务聊天机器人](#exercise-91---financial-services-chatbot)
- [练习 9.2 - Codebot](#exercise-92---codebot)

### 练习 9.1 - 金融服务聊天机器人
由于类似于法律提示的原因，金融行业内的提示也可能相当复杂。这是一个金融用例的练习，其中 Claude 用于**分析税务信息并回答问题**。就像法律服务示例一样，我们改变了一些元素的顺序，因为我们的解决方案提示对于不同的流程更有意义（但是，其他结构也可以）。

我们建议您仔细阅读变量内容（在本例中为“{QUESTION}”和“{TAX_CODE}”），以了解 Claude 需要处理哪些内容。请务必在提示中直接引用“{QUESTION}”和“{TAX_CODE}”（使用像其他示例中那样的 f 字符串语法），以便可以替换实际的变量内容。

使用与描述以及您在前面的复杂提示示例中看到的示例相匹配的内容填写提示元素字段。填写完所有要填写的提示元素后，运行该单元格以查看串联的提示以及 Claude 的响应。

请记住，提示工程很少是纯粹公式化的，尤其是对于大型且复杂的提示！开发测试用例并**尝试各种提示和提示结构以了解哪种提示最适合每种情况**非常重要。请注意，如果您*确实*更改了提示元素的顺序，您还应该记住更改“组合元素”部分中串联的顺序。

```python
######################################## INPUT VARIABLES ########################################

# First input variable - the user's question
QUESTION = "How long do I have to make an 83b election?"

# Second input variable - the tax code document that Claude will be using to answer the user's question
TAX_CODE = """
(a)General rule
If, in connection with the performance of services, property is transferred to any person other than the person for whom such services are performed, the excess of—
(1)the fair market value of such property (determined without regard to any restriction other than a restriction which by its terms will never lapse) at the first time the rights of the person having the beneficial interest in such property are transferable or are not subject to a substantial risk of forfeiture, whichever occurs earlier, over
(2)the amount (if any) paid for such property,
shall be included in the gross income of the person who performed such services in the first taxable year in which the rights of the person having the beneficial interest in such property are transferable or are not subject to a substantial risk of forfeiture, whichever is applicable. The preceding sentence shall not apply if such person sells or otherwise disposes of such property in an arm’s length transaction before his rights in such property become transferable or not subject to a substantial risk of forfeiture.
(b)Election to include in gross income in year of transfer
(1)In general
Any person who performs services in connection with which property is transferred to any person may elect to include in his gross income for the taxable year in which such property is transferred, the excess of—
(A)the fair market value of such property at the time of transfer (determined without regard to any restriction other than a restriction which by its terms will never lapse), over
(B)the amount (if any) paid for such property.
If such election is made, subsection (a) shall not apply with respect to the transfer of such property, and if such property is subsequently forfeited, no deduction shall be allowed in respect of such forfeiture.
(2)Election
An election under paragraph (1) with respect to any transfer of property shall be made in such manner as the Secretary prescribes and shall be made not later than 30 days after the date of such transfer. Such election may not be revoked except with the consent of the Secretary.

(c)Special rules
For purposes of this section—
(1)Substantial risk of forfeiture
The rights of a person in property are subject to a substantial risk of forfeiture if such person’s rights to full enjoyment of such property are conditioned upon the future performance of substantial services by any individual.

(2)Transferability of property
The rights of a person in property are transferable only if the rights in such property of any transferee are not subject to a substantial risk of forfeiture.

(3)Sales which may give rise to suit under section 16(b) of the Securities Exchange Act of 1934
So long as the sale of property at a profit could subject a person to suit under section 16(b) of the Securities Exchange Act of 1934, such person’s rights in such property are—
(A)subject to a substantial risk of forfeiture, and
(B)not transferable.
(4)For purposes of determining an individual’s basis in property transferred in connection with the performance of services, rules similar to the rules of section 72(w) shall apply.
(d)Certain restrictions which will never lapse
(1)Valuation
In the case of property subject to a restriction which by its terms will never lapse, and which allows the transferee to sell such property only at a price determined under a formula, the price so determined shall be deemed to be the fair market value of the property unless established to the contrary by the Secretary, and the burden of proof shall be on the Secretary with respect to such value.

(2)Cancellation
If, in the case of property subject to a restriction which by its terms will never lapse, the restriction is canceled, then, unless the taxpayer establishes—
(A)that such cancellation was not compensatory, and
(B)that the person, if any, who would be allowed a deduction if the cancellation were treated as compensatory, will treat the transaction as not compensatory, as evidenced in such manner as the Secretary shall prescribe by regulations,
the excess of the fair market value of the property (computed without regard to the restrictions) at the time of cancellation over the sum of—
(C)the fair market value of such property (computed by taking the restriction into account) immediately before the cancellation, and
(D)the amount, if any, paid for the cancellation,
shall be treated as compensation for the taxable year in which such cancellation occurs.
(e)Applicability of section
This section shall not apply to—
(1)a transaction to which section 421 applies,
(2)a transfer to or from a trust described in section 401(a) or a transfer under an annuity plan which meets the requirements of section 404(a)(2),
(3)the transfer of an option without a readily ascertainable fair market value,
(4)the transfer of property pursuant to the exercise of an option with a readily ascertainable fair market value at the date of grant, or
(5)group-term life insurance to which section 79 applies.
(f)Holding period
In determining the period for which the taxpayer has held property to which subsection (a) applies, there shall be included only the period beginning at the first time his rights in such property are transferable or are not subject to a substantial risk of forfeiture, whichever occurs earlier.

(g)Certain exchanges
If property to which subsection (a) applies is exchanged for property subject to restrictions and conditions substantially similar to those to which the property given in such exchange was subject, and if section 354, 355, 356, or 1036 (or so much of section 1031 as relates to section 1036) applied to such exchange, or if such exchange was pursuant to the exercise of a conversion privilege—
(1)such exchange shall be disregarded for purposes of subsection (a), and
(2)the property received shall be treated as property to which subsection (a) applies.
(h)Deduction by employer
In the case of a transfer of property to which this section applies or a cancellation of a restriction described in subsection (d), there shall be allowed as a deduction under section 162, to the person for whom were performed the services in connection with which such property was transferred, an amount equal to the amount included under subsection (a), (b), or (d)(2) in the gross income of the person who performed such services. Such deduction shall be allowed for the taxable year of such person in which or with which ends the taxable year in which such amount is included in the gross income of the person who performed such services.

(i)Qualified equity grants
(1)In general
For purposes of this subtitle—
(A)Timing of inclusion
If qualified stock is transferred to a qualified employee who makes an election with respect to such stock under this subsection, subsection (a) shall be applied by including the amount determined under such subsection with respect to such stock in income of the employee in the taxable year determined under subparagraph (B) in lieu of the taxable year described in subsection (a).

(B)Taxable year determined
The taxable year determined under this subparagraph is the taxable year of the employee which includes the earliest of—
(i)the first date such qualified stock becomes transferable (including, solely for purposes of this clause, becoming transferable to the employer),
(ii)the date the employee first becomes an excluded employee,
(iii)the first date on which any stock of the corporation which issued the qualified stock becomes readily tradable on an established securities market (as determined by the Secretary, but not including any market unless such market is recognized as an established securities market by the Secretary for purposes of a provision of this title other than this subsection),
(iv)the date that is 5 years after the first date the rights of the employee in such stock are transferable or are not subject to a substantial risk of forfeiture, whichever occurs earlier, or
(v)the date on which the employee revokes (at such time and in such manner as the Secretary provides) the election under this subsection with respect to such stock.
(2)Qualified stock
(A)In general
For purposes of this subsection, the term “qualified stock” means, with respect to any qualified employee, any stock in a corporation which is the employer of such employee, if—
(i)such stock is received—
(I)in connection with the exercise of an option, or
(II)in settlement of a restricted stock unit, and
(ii)such option or restricted stock unit was granted by the corporation—
(I)in connection with the performance of services as an employee, and
(II)during a calendar year in which such corporation was an eligible corporation.
(B)Limitation
The term “qualified stock” shall not include any stock if the employee may sell such stock to, or otherwise receive cash in lieu of stock from, the corporation at the time that the rights of the employee in such stock first become transferable or not subject to a substantial risk of forfeiture.

(C)Eligible corporation
For purposes of subparagraph (A)(ii)(II)—
(i)In general
The term “eligible corporation” means, with respect to any calendar year, any corporation if—
(I)no stock of such corporation (or any predecessor of such corporation) is readily tradable on an established securities market (as determined under paragraph (1)(B)(iii)) during any preceding calendar year, and
(II)such corporation has a written plan under which, in such calendar year, not less than 80 percent of all employees who provide services to such corporation in the United States (or any possession of the United States) are granted stock options, or are granted restricted stock units, with the same rights and privileges to receive qualified stock.
(ii)Same rights and privileges
For purposes of clause (i)(II)—
(I)except as provided in subclauses (II) and (III), the determination of rights and privileges with respect to stock shall be made in a similar manner as under section 423(b)(5),
(II)employees shall not fail to be treated as having the same rights and privileges to receive qualified stock solely because the number of shares available to all employees is not equal in amount, so long as the number of shares available to each employee is more than a de minimis amount, and
(III)rights and privileges with respect to the exercise of an option shall not be treated as the same as rights and privileges with respect to the settlement of a restricted stock unit.
(iii)Employee
For purposes of clause (i)(II), the term “employee” shall not include any employee described in section 4980E(d)(4) or any excluded employee.

(iv)Special rule for calendar years before 2018
In the case of any calendar year beginning before January 1, 2018, clause (i)(II) shall be applied without regard to whether the rights and privileges with respect to the qualified stock are the same.

(3)Qualified employee; excluded employee
For purposes of this subsection—
(A)In general
The term “qualified employee” means any individual who—
(i)is not an excluded employee, and
(ii)agrees in the election made under this subsection to meet such requirements as are determined by the Secretary to be necessary to ensure that the withholding requirements of the corporation under chapter 24 with respect to the qualified stock are met.
(B)Excluded employee
The term “excluded employee” means, with respect to any corporation, any individual—
(i)who is a 1-percent owner (within the meaning of section 416(i)(1)(B)(ii)) at any time during the calendar year or who was such a 1 percent owner at any time during the 10 preceding calendar years,
(ii)who is or has been at any prior time—
(I)the chief executive officer of such corporation or an individual acting in such a capacity, or
(II)the chief financial officer of such corporation or an individual acting in such a capacity,
(iii)who bears a relationship described in section 318(a)(1) to any individual described in subclause (I) or (II) of clause (ii), or
(iv)who is one of the 4 highest compensated officers of such corporation for the taxable year, or was one of the 4 highest compensated officers of such corporation for any of the 10 preceding taxable years, determined with respect to each such taxable year on the basis of the shareholder disclosure rules for compensation under the Securities Exchange Act of 1934 (as if such rules applied to such corporation).
(4)Election
(A)Time for making election
An election with respect to qualified stock shall be made under this subsection no later than 30 days after the first date the rights of the employee in such stock are transferable or are not subject to a substantial risk of forfeiture, whichever occurs earlier, and shall be made in a manner similar to the manner in which an election is made under subsection (b).

(B)Limitations
No election may be made under this section with respect to any qualified stock if—
(i)the qualified employee has made an election under subsection (b) with respect to such qualified stock,
(ii)any stock of the corporation which issued the qualified stock is readily tradable on an established securities market (as determined under paragraph (1)(B)(iii)) at any time before the election is made, or
(iii)such corporation purchased any of its outstanding stock in the calendar year preceding the calendar year which includes the first date the rights of the employee in such stock are transferable or are not subject to a substantial risk of forfeiture, unless—
(I)not less than 25 percent of the total dollar amount of the stock so purchased is deferral stock, and
(II)the determination of which individuals from whom deferral stock is purchased is made on a reasonable basis.
(C)Definitions and special rules related to limitation on stock redemptions
(i)Deferral stock
For purposes of this paragraph, the term “deferral stock” means stock with respect to which an election is in effect under this subsection.

(ii)Deferral stock with respect to any individual not taken into account if individual holds deferral stock with longer deferral period
Stock purchased by a corporation from any individual shall not be treated as deferral stock for purposes of subparagraph (B)(iii) if such individual (immediately after such purchase) holds any deferral stock with respect to which an election has been in effect under this subsection for a longer period than the election with respect to the stock so purchased.

(iii)Purchase of all outstanding deferral stock
The requirements of subclauses (I) and (II) of subparagraph (B)(iii) shall be treated as met if the stock so purchased includes all of the corporation’s outstanding deferral stock.

(iv)Reporting
Any corporation which has outstanding deferral stock as of the beginning of any calendar year and which purchases any of its outstanding stock during such calendar year shall include on its return of tax for the taxable year in which, or with which, such calendar year ends the total dollar amount of its outstanding stock so purchased during such calendar year and such other information as the Secretary requires for purposes of administering this paragraph.

(5)Controlled groups
For purposes of this subsection, all persons treated as a single employer under section 414(b) shall be treated as 1 corporation.

(6)Notice requirement
Any corporation which transfers qualified stock to a qualified employee shall, at the time that (or a reasonable period before) an amount attributable to such stock would (but for this subsection) first be includible in the gross income of such employee—
(A)certify to such employee that such stock is qualified stock, and
(B)notify such employee—
(i)that the employee may be eligible to elect to defer income on such stock under this subsection, and
(ii)that, if the employee makes such an election—
(I)the amount of income recognized at the end of the deferral period will be based on the value of the stock at the time at which the rights of the employee in such stock first become transferable or not subject to substantial risk of forfeiture, notwithstanding whether the value of the stock has declined during the deferral period,
(II)the amount of such income recognized at the end of the deferral period will be subject to withholding under section 3401(i) at the rate determined under section 3402(t), and
(III)the responsibilities of the employee (as determined by the Secretary under paragraph (3)(A)(ii)) with respect to such withholding.
(7)Restricted stock units
This section (other than this subsection), including any election under subsection (b), shall not apply to restricted stock units.
"""



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = ""

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = ""

##### Prompt element 4: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = ""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = ""

##### Prompt element 6: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = ""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = ""

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = ""

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = ""

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = ""



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

❓ 如果您想查看可能的解决方案，请运行下面的单元格！

```python
from hints import exercise_9_1_solution; print(exercise_9_1_solution)
```

### 练习 9.2 - Codebot
在本练习中，我们将编写一个**编码辅助和教学机器人的提示，该机器人读取代码并在适当时提供指导性更正**。使用与描述以及您在前面的复杂提示示例中看到的示例相匹配的内容填写提示元素字段。填写完所有要填写的提示元素后，运行该单元格以查看串联的提示以及 Claude 的响应。

我们建议您仔细阅读变量内容（在本例中为“{CODE}”），以了解 Claude 需要使用哪些内容。请务必在提示中直接引用“{CODE}”（使用像其他示例中那样的 f 字符串语法），以便可以替换实际的变量内容。

```python
######################################## INPUT VARIABLES ########################################

# Input variable - the code that Claude needs to read and assist the user with correcting
CODE = """
# Function to print multiplicative inverses
def print_multiplicative_inverses(x, n):
  for i in range(n):
    print(x / i)
"""



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = ""

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = ""

##### Prompt element 4: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = ""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = ""

##### Prompt element 6: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = ""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = ""

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = ""

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = ""

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = ""



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

❓ 如果您想查看可能的解决方案，请运行下面的单元格！

```python
from hints import exercise_9_2_solution; print(exercise_9_2_solution)
```

---

## 恭喜并下一步！

如果您完成了所有练习，**您现在已跻身 LLM 窃听者前 0.1% 之列**。精英之一！

您所学到的技术，从逐步思考到分配角色，再到使用示例，再到一般性的全面清晰的写作，可以**以无数种方式进行合并、重新混合和调整**。

快速工程是一门非常新的学科，因此请保持开放的态度。您可能会发现下一个伟大的提示技巧。

如果您想查看**更多好的提示示例**以获取灵感：
- 从我们的 [cookbook](https://anthropic.com/cookbook) 中学习生产就绪提示的示例
- 通读我们的[提示指南](https://docs.anthropic.com/claude/docs/prompt-engineering)
- 查看我们的[提示库](https://anthropic.com/prompts) 获取灵感
- 尝试我们的实验性 [metaprompt](https://docs.anthropic.com/claude/docs/helper-metaprompt-experimental) 让 Claude 为您编写提示模板！
- 在我们的 [discord 服务器](https://anthropic.com/discord) 中提问
- 了解 [Anthropic API 参数](https://docs.anthropic.com/claude/reference/complete_post)，例如温度和 `max_tokens`
- 如果您感觉学术性强，请阅读一些有关即时工程的[论文](https://www.promptingguide.ai/papers)
- 练习构建提示，让克劳德做你感兴趣的事情

如果您想了解本教程范围之外的一些真正高级的提示技术，请单击附录！但首先，运行下面的单元格。

```python
# Prompt
PROMPT = "Write an ode to a fabulous student who has just completed a course on prompt engineering, in the form of a sonnet."

# Print Claude's response
print(get_completion(PROMPT))
```

---

## 游乐场示例

您可以在此区域自由地尝试本课程中显示的提示示例，并调整提示以了解它如何影响 Claude 的响应。

```python
######################################## INPUT VARIABLES ########################################

# First input variable - the conversation history (this can also be added as preceding `user` and `assistant` messages in the API call)
HISTORY = """Customer: Give me two possible careers for sociology majors.

Joe: Here are two potential careers for sociology majors:

- Social worker - Sociology provides a strong foundation for understanding human behavior and social systems. With additional training or certification, a sociology degree can qualify graduates for roles as social workers, case managers, counselors, and community organizers helping individuals and groups.

- Human resources specialist - An understanding of group dynamics and organizational behavior from sociology is applicable to careers in human resources. Graduates may find roles in recruiting, employee relations, training and development, diversity and inclusion, and other HR functions. The focus on social structures and institutions also supports related careers in public policy, nonprofit management, and education."""

# Second input variable - the user's question
QUESTION = "Which of the two careers requires more than a Bachelor's degree?"



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = "You will be acting as an AI career coach named Joe created by the company AdAstra Careers. Your goal is to give career advice to users. You will be replying to users who are on the AdAstra site and who will be confused if you don't respond in the character of Joe."

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = "You should maintain a friendly customer service tone."

##### Prompt element 4: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = """Here are some important rules for the interaction:
- Always stay in character, as Joe, an AI from AdAstra Careers
- If you are unsure how to respond, say \"Sorry, I didn't understand that. Could you rephrase your question?\"
- If someone asks something irrelevant, say, \"Sorry, I am Joe and I give career advice. Do you have a career question today I can help you with?\""""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = """Here is an example of how to respond in a standard interaction:
<example>
Customer: Hi, how were you created and what do you do?
Joe: Hello! My name is Joe, and I was created by AdAstra Careers to give career advice. What can I help you with today?
</example>"""

##### Prompt element 6: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = f"""Here is the conversational history (between the user and you) prior to the question. It could be empty if there is no history:
<history>
{HISTORY}
</history>

Here is the user's question:
<question>
{QUESTION}
</question>"""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = "How do you respond to the user's question?"

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = "Think about your answer first before you respond."

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = "Put your response in <response></response> tags."

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = "[Joe] <response>"



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```

```python
######################################## INPUT VARIABLES ########################################

# First input variable - the legal document
LEGAL_RESEARCH = """<search_results>
<search_result id=1>
The animal health industry became caught up in a number of patent and trademark lawsuits during the past year. In 1994, Barclay Slocum obtained patents for the tibial plateau leveling osteotomy procedure, which is used in the treatment of dogs with cranial cruciate ligament rupture, and for the devices used in the procedure. During 2006, Slocum Enterprises filed a patent infringement suit against New Generation Devices, arguing that the Unity Cruciate Plate manufactured by New Generation infringed on the patent for the Slocum TPLO plate. However, the court never reached a decision on the issue of patent infringement, ruling that it did not have jurisdiction on the basis of the small number of plates sold in the state in which the case was filed and the information provided on a Web site maintained by Slocum Enterprises. Other patent battles waged during 2006 concerned the use of laser technology for onychectomy in cats, pet identification chips, pig vaccines, and pet “deshedding” tools.
</search_result>
<search_result id=2>
In Canada, the British Columbia Veterinary Medical Association brought suit against a nonveterinarian, claiming that he engaged in cutting or otherwise removing hooks from horses' teeth and floating horses' teeth with power and manual tools, provided advice and diagnoses in return for a fee, and held himself out as being qualified and willing to provide treatment with respect to these activities. The court held that the intention of the legislature in passing the Veterinary Profession Act was the protection of the public and animals and further held that monopolistic statutes serve the purpose of protecting the public. In addition, the court concluded that dentistry, at its core, relates to the health of the teeth and gums; is distinct from cosmetic and other types of care of animals; and, therefore, falls under the definition of the practice of veterinary medicine. The nonveterinarian was enjoined from providing services without a veterinarian supervising the procedures.
</search_result>
<search_result id=3>
The aftermath of Hurricane Katrina, which hit the Gulf Coast of the United States during 2005, spurred changes to the way animals are treated during natural disasters. In 2006, Hawaii, Louisiana, and New Hampshire all enacted laws that address issues regarding the care of animals during disasters, such as providing shelters for pets and allowing service animals to be kept with the people they serve. In addition, Congress passed, and the President signed, the Pet Evacuation and Transportation Standards Act during 2006, which requires state and local emergency preparedness authorities to include in their evacuation plans information on how they will accommodate household pets and service animals in case of a disaster. California passed a law that will require its Office of Emergency Services, Department of Agriculture, and other agencies involved with disaster response preparation to develop a plan for the needs of service animals, livestock, equids, and household pets in the event of a disaster or major emergency.
</search_result>
</search_results>"""

# Second input variable - the user's question
QUESTION = "Are there any laws about what to do with pets during a hurricane?"



######################################## PROMPT ELEMENTS ########################################

##### Prompt element 1: `user` role
# Make sure that your Messages API call always starts with a `user` role in the messages array.
# The get_completion() function as defined above will automatically do this for you.

##### Prompt element 2: Task context
# Give Claude context about the role it should take on or what goals and overarching tasks you want it to undertake with the prompt.
# It's best to put context early in the body of the prompt.
TASK_CONTEXT = "You are an expert lawyer."

##### Prompt element 3: Tone context
# If important to the interaction, tell Claude what tone it should use.
# This element may not be necessary depending on the task.
TONE_CONTEXT = ""

##### Prompt element 4: Input data to process
# If there is data that Claude needs to process within the prompt, include it here within relevant XML tags.
# Feel free to include multiple pieces of data, but be sure to enclose each in its own set of XML tags.
# This element may not be necessary depending on task. Ordering is also flexible.
INPUT_DATA = f"""Here is some research that's been compiled. Use it to answer a legal question from the user.
<legal_research>
{LEGAL_RESEARCH}
</legal_research>"""

##### Prompt element 5: Examples
# Provide Claude with at least one example of an ideal response that it can emulate. Encase this in <example></example> XML tags. Feel free to provide multiple examples.
# If you do provide multiple examples, give Claude context about what it is an example of, and enclose each example in its own set of XML tags.
# Examples are probably the single most effective tool in knowledge work for getting Claude to behave as desired.
# Make sure to give Claude examples of common edge cases. If your prompt uses a scratchpad, it's effective to give examples of how the scratchpad should look.
# Generally more examples = better.
EXAMPLES = """When citing the legal research in your answer, please use brackets containing the search index ID, followed by a period. Put these at the end of the sentence that's doing the citing. Examples of proper citation format:

<examples>
<example>
The statute of limitations expires after 10 years for crimes like this. [3].
</example>
<example>
However, the protection does not apply when it has been specifically waived by both parties. [5].
</example>
</examples>"""

##### Prompt element 6: Detailed task description and rules
# Expand on the specific tasks you want Claude to do, as well as any rules that Claude might have to follow.
# This is also where you can give Claude an "out" if it doesn't have an answer or doesn't know.
# It's ideal to show this description and rules to a friend to make sure it is laid out logically and that any ambiguous words are clearly defined.
TASK_DESCRIPTION = """Write a clear, concise answer to this question:

<question>
{QUESTION}
</question>

It should be no more than a couple of paragraphs. If possible, it should conclude with a single sentence directly answering the user's question. However, if there is not sufficient information in the compiled research to produce such an answer, you may demur and write "Sorry, I do not have sufficient information at hand to answer this question."."""

##### Prompt element 7: Immediate task description or request #####
# "Remind" Claude or tell Claude exactly what it's expected to immediately do to fulfill the prompt's task.
# This is also where you would put in additional variables like the user's question.
# It generally doesn't hurt to reiterate to Claude its immediate task. It's best to do this toward the end of a long prompt.
# This will yield better results than putting this at the beginning.
# It is also generally good practice to put the user's query close to the bottom of the prompt.
IMMEDIATE_TASK = ""

##### Prompt element 8: Precognition (thinking step by step)
# For tasks with multiple steps, it's good to tell Claude to think step by step before giving an answer
# Sometimes, you might have to even say "Before you give your answer..." just to make sure Claude does this first.
# Not necessary with all prompts, though if included, it's best to do this toward the end of a long prompt and right after the final immediate task request or description.
PRECOGNITION = "Before you answer, pull out the most relevant quotes from the research in <relevant_quotes> tags."

##### Prompt element 9: Output formatting
# If there is a specific way you want Claude's response formatted, clearly tell Claude what that format is.
# This element may not be necessary depending on the task.
# If you include it, putting it toward the end of the prompt is better than at the beginning.
OUTPUT_FORMATTING = "Put your two-paragraph response in <answer> tags."

##### Prompt element 10: Prefilling Claude's response (if any)
# A space to start off Claude's answer with some prefilled words to steer Claude's behavior or response.
# If you want to prefill Claude's response, you must put this in the `assistant` role in the API call.
# This element may not be necessary depending on the task.
PREFILL = "<relevant_quotes>"



######################################## COMBINE ELEMENTS ########################################

PROMPT = ""

if TASK_CONTEXT:
    PROMPT += f"""{TASK_CONTEXT}"""

if TONE_CONTEXT:
    PROMPT += f"""\n\n{TONE_CONTEXT}"""

if INPUT_DATA:
    PROMPT += f"""\n\n{INPUT_DATA}"""

if EXAMPLES:
    PROMPT += f"""\n\n{EXAMPLES}"""

if TASK_DESCRIPTION:
    PROMPT += f"""\n\n{TASK_DESCRIPTION}"""

if IMMEDIATE_TASK:
    PROMPT += f"""\n\n{IMMEDIATE_TASK}"""

if PRECOGNITION:
    PROMPT += f"""\n\n{PRECOGNITION}"""

if OUTPUT_FORMATTING:
    PROMPT += f"""\n\n{OUTPUT_FORMATTING}"""

# Print full prompt
print("--------------------------- Full prompt with variable substutions ---------------------------")
print("USER TURN")
print(PROMPT)
print("\nASSISTANT TURN")
print(PREFILL)
print("\n------------------------------------- Claude's response -------------------------------------")
print(get_completion(PROMPT, prefill=PREFILL))
```
