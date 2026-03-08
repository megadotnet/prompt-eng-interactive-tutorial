# 教程操作方法

本教程**需要 API 密钥**才能进行交互。如果您没有 API 密钥，可以通过 [Anthropic 控制台](https://console.anthropic.com/) 注册一个，或查看我们的[静态教程答案](https://docs.google.com/spreadsheets/u/0/d/1jIxjzUWG-6xBVIa2ay6yDpLyeuOh_hR_ZB75a47KX_E/edit)。

## 如何开始

1. 将此存储库克隆到您的本地计算机。

2. 通过运行以下命令安装所需的依赖项：

```python
!pip install anthropic
```

3. 设置您的 API 密钥和型号名称。将 `"your_api_key_here"` 替换为您实际的 Anthropic API 密钥。

```python
API_KEY = "your_api_key_here"
MODEL_NAME = "claude-3-haiku-20240307"

# Stores the API_KEY & MODEL_NAME variables for use across notebooks within the IPython store
%store API_KEY
%store MODEL_NAME
```

4. 按照提供的说明按顺序运行笔记本单元。

---

## 使用说明和技巧 💡

- 本课程使用 Claude 3 Haiku，温度为 0。我们将在课程后面详细讨论温度。目前，只要了解这些设置会产生更具确定性的结果就足够了。本课程中的所有即时工程技术也适用于上一代传统 Claude 模型，例如 Claude 2 和 Claude Instant 1.2。

- 您可以使用“Shift + Enter”执行单元格并移至下一个单元格。

- 当您到达教程页面的底部时，导航到文件夹中的下一个编号的文件，或者如果您已完成该章节文件中的内容，则导航到下一个编号的文件夹。

### Anthropic SDK 和消息 API
在本教程中，我们将使用 [Anthropic python SDK](https://docs.anthropic.com/claude/reference/client-sdks) 和 [Messages API](https://docs.anthropic.com/claude/reference/messages_post)。

下面是本教程中运行提示的示例。首先，我们创建“get_completion”，这是一个辅助函数，用于向 Claude 发送提示并返回 Claude 生成的响应。现在运行该单元格。

```python
import anthropic

client = anthropic.Anthropic(api_key=API_KEY)

def get_completion(prompt: str):
    message = client.messages.create(
        model=MODEL_NAME,
        max_tokens=2000,
        temperature=0.0,
        messages=[
          {"role": "user", "content": prompt}
        ]
    )
    return message.content[0].text
```

现在，我们将为 Claude 编写一个示例提示，并通过运行“get_completion”辅助函数来打印 Claude 的输出。运行下面的单元格将打印出下面克劳德的响应。

请随意使用提示字符串来引发 Claude 的不同反应。

```python
# Prompt
prompt = "Hello, Claude!"

# Get Claude's response
print(get_completion(prompt))
```

前面定义的“API_KEY”和“MODEL_NAME”变量将在整个教程中使用。只需确保从上到下运行每个教程页面的单元格即可。
