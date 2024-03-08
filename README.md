# ShellGPT
一个由AI大型语言模型（LLM）驱动的命令行生产力工具。该命令行工具提供了对**shell命令、代码片段、文档**的简化生成，消除了对外部资源（如Google搜索）的需求。支持Linux、macOS、Windows，并与PowerShell、CMD、Bash、Zsh等主要Shell兼容。

https://github.com/TheR1D/shell_gpt/assets/16740832/9197283c-db6a-4b46-bfea-3eb776dd9093

## 安装
```shell
pip install shell-gpt
```
默认情况下，ShellGPT使用OpenAI的API和GPT-4模型。您需要一个API密钥，您可以在[这里](https://beta.openai.com/account/api-keys)生成一个。您将被要求输入您的密钥，然后它将被存储在`~/.config/shell_gpt/.sgptrc`中。OpenAI API并非免费，有关更多信息，请参阅[OpenAI定价](https://openai.com/pricing)。

> [!TIP]
> 或者，您可以使用本地托管的开源模型，这些模型可免费使用。要使用本地模型，您需要运行自己的LLM后端服务器，例如[Ollama](https://github.com/ollama/ollama)。要使用Ollama设置ShellGPT，请按照这个详细的[指南](https://github.com/TheR1D/shell_gpt/wiki/Ollama)进行操作。
>
> **❗️请注意，ShellGPT并未针对本地模型进行优化，可能无法按预期工作。**

## 使用方法
**ShellGPT**旨在快速分析和检索信息。它对于从技术配置到一般知识的简单请求非常有用。
```shell
sgpt "什么是斐波那契数列"
# -> 斐波那契数列是一系列数字，其中每个数字 ...
```

ShellGPT可以接受来自标准输入和命令行参数的提示。无论您是喜欢通过终端进行输入管道，还是直接指定参数，`sgpt`都可以满足您的需求。例如，您可以根据差异轻松生成一个git提交消息：
```shell
git diff | sgpt "为我的更改生成git提交消息"
# -> 将主要功能详细信息添加到README.md中
```

您可以通过使用stdin传递来自各种来源的日志，以及一个提示来分析日志。例如，我们可以使用它快速分析日志，识别错误并获得可能的解决方案建议：
```shell
docker logs -n 20 my_app | sgpt "检查日志，查找错误，提供可能的解决方案"
```
```text
检测到错误：第7行连接超时。
可能的解决方案：检查网络连接和防火墙设置。
检测到错误：第12行内存分配失败。
可能的解决方案：考虑增加内存分配或优化应用程序内存使用。
```

您还可以使用各种重定向运算符传递输入：
```shell
sgpt "总结" < document.txt
# -> 该文档讨论了影响...
sgpt << EOF
什么是学习Golang的最佳方法？
提供一个简单的hello world示例。
EOF
# -> 学习Golang的最佳方法是...
sgpt <<< "什么是学习shell重定向的最佳方式？"
# -> 学习shell重定向的最佳方式是通过...
```

当然！使用Shell GPT，您可以快速生成常见的shell命令。只需使用`--shell`或`-s`选项，即可在终端中生成和执行所需的命令。

```shell
sgpt --shell "在当前文件夹中查找所有json文件"
# -> find . -type f -name "*.json"
# -> [E]xecute, [D]escribe, [A]bort: e
```

Shell GPT会根据您使用的操作系统和`$SHELL`环境变量提供特定系统的shell命令。例如，如果您要求`sgpt`更新您的系统，它将返回基于您的操作系统的命令。下面是在macOS上的示例：

```shell
sgpt -s "更新我的系统"
# -> sudo softwareupdate -i -a
# -> [E]xecute, [D]escribe, [A]bort: e
```

同样的提示，在Ubuntu上使用时，将生成不同的建议：

```shell
sgpt -s "更新我的系统"
# -> sudo apt update && sudo apt upgrade -y
# -> [E]xecute, [D]escribe, [A]bort: e
```

让我们尝试一下Docker：

```shell
sgpt -s "启动nginx容器，挂载./index.html"
# -> docker run -d -p 80:80 -v $(pwd)/index.html:/usr/share/nginx/html/index.html nginx
# -> [E]xecute, [D]escribe, [A]bort: e
```

我们仍然可以使用管道将输入传递给`sgpt`并生成shell命令：

```shell
sgpt -s "使用以下数据在localhost上进行POST请求" < data.json
# -> curl -X POST -H "Content-Type: application/json" -d '{"a": 1, "b": 2}' http://localhost
# -> [E]xecute, [D]escribe, [A]bort: e
```

在提示中应用额外的shell技巧，例如将文件名传递给`ffmpeg`：

```shell
ls
# -> 1.mp4 2.mp4 3.mp4
sgpt -s "使用$(ls -m)将多个视频文件合并为一个没有音频的视频文件。"
# -> ffmpeg -i 1.mp4 -i 2.mp4 -i 3.mp4 -filter_complex "[0:v] [1:v] [2:v] concat=n=3:v=1 [v]" -map "[v]" out.mp4
# -> [E]xecute, [D]escribe, [A]bort: e
```

如果您想使用管道传递生成的shell命令，可以使用`--no-interaction`选项。这将禁用交互模式，并将生成的命令打印到stdout。在此示例中，我们使用`pbcopy`将生成的命令复制到剪贴板：

```shell
sgpt -s "在当前文件夹中查找所有json文件" --no-interaction | pbcopy
```


### Shell integration
这是一个非常方便的功能，它允许您在终端中直接使用 `sgpt` 的命令补全，无需输入 `sgpt` 的提示符和参数。Shell 集成功能支持使用热键在终端中使用 ShellGPT，支持 Bash 和 ZSH 两种 shell。该功能将 `sgpt` 的命令补全直接放入终端缓冲区（输入行），可以立即编辑建议的命令。

https://github.com/TheR1D/shell_gpt/assets/16740832/bead0dab-0dd9-436d-88b7-6abfb2c556c1

要安装 Shell 集成，运行 `sgpt --install-integration` 并重新启动终端以应用更改。这将在您的 `.bashrc` 或 `.zshrc` 文件中添加几行代码。之后，您可以使用 `Ctrl+l`（默认情况下）来调用 ShellGPT。按下 `Ctrl+l` 后，它将用建议的命令替换当前的输入行（缓冲区）。您可以编辑它，然后按下 `Enter` 来执行命令。

### 生成代码
通过使用 `--code` 或 `-c` 参数，您可以明确请求纯代码输出，例如：
```shell
sgpt --code "使用 Python 解决 Fizz Buzz 问题"
```

```python
for i in range(1, 101):
    if i % 3 == 0 and i % 5 == 0:
        print("FizzBuzz")
    elif i % 3 == 0:
        print("Fizz")
    elif i % 5 == 0:
        print("Buzz")
    else:
        print(i)
```
由于这是有效的 Python 代码，我们可以将其重定向到一个文件中：
```shell
sgpt --code "使用 Python 解决经典 Fizz Buzz 问题" > fizz_buzz.py
python fizz_buzz.py
# 1
# 2
# Fizz
# 4
# Buzz
# ...
```

我们还可以使用管道传递输入：
```shell
cat fizz_buzz.py | sgpt --code "为我的代码的每一行生成注释"
```
```python
# 循环遍历从1到100的数字
for i in range(1, 101):
    # 检查数字是否既能被3整除又能被5整除
    if i % 3 == 0 and i % 5 == 0:
        # 如果数字能被3和5同时整除，打印"FizzBuzz"
        print("FizzBuzz")
    # 检查数字是否能被3整除
    elif i % 3 == 0:
        # 如果数字能被3整除，打印"Fizz"
        print("Fizz")
    # 检查数字是否能被5整除
    elif i % 5 == 0:
        # 如果数字能被5整除，打印"Buzz"
        print("Buzz")
    # 如果数字既不能被3整除也不能被5整除，打印数字本身
    else:
        print(i)
```

### 聊天模式 
经常需要保留和回顾对话。`sgpt`会在每次请求LLM完成时创建对话式对话。对话可以逐步发展（聊天模式）或以REPL循环的方式进行交互（REPL模式）。两种方式都依赖于同一个称为对话会话的基础对象，该会话位于[可配置](#运行时配置文件)的`CHAT_CACHE_PATH`位置。

要开始对话，请使用`--chat`选项，后面跟着一个唯一的会话名称和一个提示。
```shell
sgpt --chat conversation_1 "请记住我的最喜欢的数字：4"
# -> 我将记住你最喜欢的数字是4。
sgpt --chat conversation_1 "我的最喜欢的数字加上4会是多少？"
# -> 你最喜欢的数字是4，所以如果我们加上4，结果将是8。
```

您可以使用聊天会话逐步改进GPT的建议，提供额外的细节。可以使用`--code`或`--shell`选项来启动`--chat`：
```shell
sgpt --chat conversation_2 --code "使用Python向localhost发送请求"
```
```python
import requests

response = requests.get('http://localhost')
print(response.text)
```

让我们请求LLM为我们的请求添加缓存：
```shell
sgpt --chat conversation_2 --code "添加缓存"
```
```python
import requests
from cachecontrol import CacheControl

sess = requests.session()
cached_sess = CacheControl(sess)

response = cached_sess.get('http://localhost')
print(response.text)
```

对于shell命令也适用：
```shell
sgpt --chat conversation_3 --shell "当前文件夹中的内容是什么"
# -> ls
sgpt --chat conversation_3 "按名称排序"
# -> ls | sort
sgpt --chat conversation_3 "使用FFMPEG将它们连接起来"
# -> ffmpeg -i "concat:$(ls | sort | tr '\n' '|')" -codec copy output.mp4
sgpt --chat conversation_3 "将生成的文件转换为MP3格式"
# -> ffmpeg -i output.mp4 -vn -acodec libmp3lame -ac 2 -ab 160k -ar 48000 final_output.mp3
```

要列出任一对话模式中的所有会话，请使用`--list-chats`或`-lc`选项：  
```shell
sgpt --list-chats
# .../shell_gpt/chat_cache/conversation_1  
# .../shell_gpt/chat_cache/conversation_2
```

要显示与特定对话相关的所有消息，请使用`--show-chat`选项，后面跟着会话名称：
```shell
sgpt --show-chat conversation_1
# user: 请记住我的最喜欢的数字：4
# assistant: 我将记住你最喜欢的数字是4。
# user: 我的最喜欢的数字加上4会是多少？
# assistant: 你最喜欢的数字是4，所以如果我们加上4，结果将是8。
```

### REPL 模式
有一个非常方便的 REPL（读取-求值-打印循环）模式，它允许您与 GPT 模型进行交互式聊天。要在 REPL 模式下开始一个聊天会话，请使用 `--repl` 选项后面跟一个唯一的会话名称。您还可以使用 "temp" 作为会话名称来启动临时 REPL 会话。请注意，`--chat` 和 `--repl` 使用相同的底层对象，因此您可以使用 `--chat` 来启动一个聊天会话，然后使用 `--repl` 来继续在 REPL 模式下进行对话。

<p align="center">
  <img src="https://s10.gifyu.com/images/repl-demo.gif" alt="gif">
</p>

```text
sgpt --repl temp
进入 REPL 模式，按 Ctrl+C 退出。
>>> 什么是 REPL？
REPL 代表读取-求值-打印循环。它是一个编程环境...
>>> 如何在 REPL 中使用 Python？
要在 REPL 中使用 Python，您可以简单地打开终端或命令提示符...
```

REPL 模式可以与 `--shell` 和 `--code` 选项一起使用，非常方便进行交互式的 shell 命令和代码生成：
```text
sgpt --repl temp --shell
进入 shell REPL 模式，输入 [e] 执行命令，或按 Ctrl+C 退出。
>>> 当前文件夹中有什么？
ls
>>> 显示文件大小
ls -lh
>>> 按文件大小排序它们
ls -lhS
>>> e（仅输入 e 执行命令，或输入 d 进行描述）
```

要提供多行提示，请使用三个引号 `"""`：
```text
sgpt --repl temp
进入 REPL 模式，按 Ctrl+C 退出。
>>> """
... 解释以下代码：
... import random
... print(random.randint(1, 10))
... """
这是一个使用 random 模块生成和打印随机整数的 Python 脚本。
```

您还可以通过将其作为参数或标准输入甚至两者同时传递来使用初始提示进入 REPL 模式：
```shell
sgpt --repl temp < my_app.py
```
```text
进入 REPL 模式，按 Ctrl+C 退出。
─────────────────────────────────────── 输入 ────────────────────────────────────
name = input("What is your name?")
print(f"Hello {name}")
───────────────────────────────────────────────────────────────────────────────
>>> 这段代码是关于什么的？
您提供的代码片段是用 Python 编写的。它提示用户...
>>> 后续问题...
```

### 函数调用
[函数调用](https://platform.openai.com/docs/guides/function-calling)是OpenAI提供的一个强大功能。它允许LLM在您的系统中执行函数，用于完成各种任务。要安装[默认函数](https://github.com/TheR1D/shell_gpt/tree/main/sgpt/default_functions/)，请运行:
```shell
sgpt --install-functions
```

ShellGPT提供了一种方便的方法来定义和使用函数。为了创建您的自定义函数，请转到`~/.config/shell_gpt/functions`并创建一个新的.py文件，文件名为函数名。在这个文件中，您可以使用以下语法来定义您的函数:
```python
# execute_shell_command.py
import subprocess
from pydantic import Field
from instructor import OpenAISchema


class Function(OpenAISchema):
    """
    执行一个shell命令并返回输出（结果）。
    """
    shell_command: str = Field(..., example="ls -la", descriptions="要执行的shell命令。")

    class Config:
        title = "execute_shell_command"

    @classmethod
    def execute(cls, shell_command: str) -> str:
        result = subprocess.run(shell_command.split(), capture_output=True, text=True)
        return f"退出码: {result.returncode}, 输出:\n{result.stdout}"
```

类内的文档字符串注释将作为函数的描述传递给OpenAI API，还有`title`属性和参数描述。如果LLM决定使用您的函数，将调用`execute`函数。在这种情况下，我们允许LLM在我们的系统中执行任何Shell命令。由于我们返回了命令的输出，LLM将能够分析它并决定是否适合提示。以下是LLM可能执行该函数的示例:
```shell
sgpt "查看/tmp文件夹中的文件有哪些?"
# -> @FunctionCall execute_shell_command(shell_command="ls /tmp")
# -> /tmp文件夹包含以下文件和目录:
# -> test.txt
# -> test.json
```

请注意，如果由于某种原因函数（execute_shell_command）返回错误，LLM可能会根据输出尝试完成任务。假设我们的系统中没有安装`jq`，我们要求LLM解析JSON文件:
```shell
sgpt "使用jq解析/tmp/test.json文件并仅返回email值"
# -> @FunctionCall execute_shell_command(shell_command="jq -r '.email' /tmp/test.json")
# -> 看起来系统中没有安装jq。让我试着使用brew安装它。
# -> @FunctionCall execute_shell_command(shell_command="brew install jq")
# -> jq已成功安装。让我再次尝试解析文件。
# -> @FunctionCall execute_shell_command(shell_command="jq -r '.email' /tmp/test.json")
# -> /tmp/test.json中的email值是johndoe@example。
```

还可以在提示中链接多个函数调用:
```shell
sgpt "播放音乐并打开黑客新闻"
# -> @FunctionCall play_music()
# -> @FunctionCall open_url(url="https://news.ycombinator.com")
# -> 音乐正在播放，并且黑客新闻已在您的浏览器中打开。享受吧！
```

这只是一个简单的示例，介绍了如何使用函数调用。它是一个真正强大的功能，可用于完成各种复杂的任务。我们在GitHub讨论中专门设有一个[类别](https://github.com/TheR1D/shell_gpt/discussions/categories/functions)用于共享和讨论函数。
LLM可能执行破坏性命令，请自行承担风险❗️

### 角色
ShellGPT允许你创建自定义角色，用于生成代码、shell命令或满足你的特定需求。要创建新角色，使用`--create-role`选项后跟角色名称。你将被要求提供角色的描述以及其他细节。这将在`~/.config/shell_gpt/roles`目录下创建一个以角色名称命名的JSON文件。在这个目录中，你还可以编辑默认的`sgpt`角色，如**shell**、**code**和**default**。使用`--list-roles`选项列出所有可用角色，使用`--show-role`选项显示特定角色的详细信息。下面是一个自定义角色的示例：
```shell
sgpt --create-role json_generator
# 输入角色描述：只提供有效的JSON作为响应。
sgpt --role json_generator "random: user, password, email, address"
```
```json
{
  "user": "JohnDoe",
  "password": "p@ssw0rd",
  "email": "johndoe@example.com",
  "address": {
    "street": "123 Main St",
    "city": "Anytown",
    "state": "CA",
    "zip": "12345"
  }
}
```

如果角色的描述中包含"APPLY MARKDOWN"（区分大小写）一词，则聊天将以Markdown格式显示。

### 请求缓存
使用`--cache`（默认）和`--no-cache`选项来控制缓存。这个缓存适用于所有对OpenAI API的`sgpt`请求：
```shell
sgpt "what are the colors of a rainbow"
# -> The colors of a rainbow are red, orange, yellow, green, blue, indigo, and violet.
```
下次，相同的查询将从本地缓存中立即获取结果。注意，`sgpt "what are the colors of a rainbow" --temperature 0.5`将发起一个新的请求，因为我们没有在上次请求中提供`--temperature`（`--top-probability`也是如此）。

这只是一些使用OpenAI GPT模型的示例，我相信你会发现在你的特定用例中它非常有用。

### 运行时配置文件
你可以在运行时配置文件`~/.config/shell_gpt/.sgptrc`中设置一些参数：
```text
# API密钥，也可以使用OPENAI_API_KEY环境变量定义。
OPENAI_API_KEY=your_api_key
# 后端服务器的基本URL。如果为"default"，将根据--model解析URL。
API_BASE_URL=default
# 每个聊天会话中缓存的最大消息数量。
CHAT_CACHE_LENGTH=100
# 聊天缓存文件夹。
CHAT_CACHE_PATH=/tmp/shell_gpt/chat_cache
# 请求缓存长度（数量）。
CACHE_LENGTH=100
# 请求缓存文件夹。
CACHE_PATH=/tmp/shell_gpt/cache
# 请求超时时间（秒）。
REQUEST_TIMEOUT=60
# 默认使用的OpenAI模型。
DEFAULT_MODEL=gpt-3.5-turbo
# shell和code完成的默认颜色。
DEFAULT_COLOR=magenta
# 当处于--shell模式时，默认为不输入。
DEFAULT_EXECUTE_SHELL_CMD=false
# 禁用响应的流式传输
DISABLE_STREAMING=false
# 用于查看Markdown的pygment主题（默认/描述角色）。
CODE_THEME=default
# 包含函数的目录路径。
OPENAI_FUNCTIONS_PATH=/Users/user/.config/shell_gpt/functions
# 使用函数时打印函数的输出。
SHOW_FUNCTIONS_OUTPUT=false
# 允许LLM使用函数。
OPENAI_USE_FUNCTIONS=true
# 强制使用LiteLLM（用于本地LLM）。
USE_LITELLM=false
```
`DEFAULT_COLOR`的可能选项：black、red、green、yellow、blue、magenta、cyan、white、bright_black、bright_red、bright_green、bright_yellow、bright_blue、bright_magenta、bright_cyan、bright_white。
`CODE_THEME`的可能选项：https://pygments.org/styles/

### 参数的完整列表
```text
╭─ Arguments ──────────────────────────────────────────────────────────────────────────────────────────────╮
│   prompt      [PROMPT]  The prompt to generate completions for.                                          │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Options ────────────────────────────────────────────────────────────────────────────────────────────────╮
│ --model            TEXT                       Large language model to use. [default: gpt-4-1106-preview] │
│ --temperature      FLOAT RANGE [0.0<=x<=2.0]  Randomness of generated output. [default: 0.0]             │
│ --top-p            FLOAT RANGE [0.0<=x<=1.0]  Limits highest probable tokens (words). [default: 1.0]     │
│ --md             --no-md                      Prettify markdown output. [default: md]                    │
│ --editor                                      Open $EDITOR to provide a prompt. [default: no-editor]     │
│ --cache                                       Cache completion results. [default: cache]                 │
│ --version                                     Show version.                                              │
│ --help                                        Show this message and exit.                                │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Assistance Options ─────────────────────────────────────────────────────────────────────────────────────╮
│ --shell           -s                      Generate and execute shell commands.                           │
│ --interaction         --no-interaction    Interactive mode for --shell option. [default: interaction]    │
│ --describe-shell  -d                      Describe a shell command.                                      │
│ --code            -c                      Generate only code.                                            │
│ --functions           --no-functions      Allow function calls. [default: functions]                     │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Chat Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│ --chat                 TEXT  Follow conversation with id, use "temp" for quick session. [default: None]  │
│ --repl                 TEXT  Start a REPL (Read–eval–print loop) session. [default: None]                │
│ --show-chat            TEXT  Show all messages from provided chat id. [default: None]                    │
│ --list-chats  -lc            List all existing chat ids.                                                 │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
╭─ Role Options ───────────────────────────────────────────────────────────────────────────────────────────╮
│ --role                  TEXT  System role for GPT model. [default: None]                                 │
│ --create-role           TEXT  Create role. [default: None]                                               │
│ --show-role             TEXT  Show role. [default: None]                                                 │
│ --list-roles   -lr            List roles.                                                                │
╰──────────────────────────────────────────────────────────────────────────────────────────────────────────╯
```

## Docker
使用 `OPENAI_API_KEY` 环境变量以及容器卷来运行容器以存储缓存:
```shell
docker run --rm \
           --env OPENAI_API_KEY="你的 OPENAI API 密钥" \
           --volume gpt-cache:/tmp/shell_gpt \
       ghcr.io/ther1d/shell_gpt --chat rainbow "what are the colors of a rainbow"
```

使用别名和 `OPENAI_API_KEY` 环境变量的对话示例:
```shell
alias sgpt="docker run --rm --env OPENAI_API_KEY --volume gpt-cache:/tmp/shell_gpt ghcr.io/ther1d/shell_gpt"
export OPENAI_API_KEY="你的 OPENAI API 密钥"
sgpt --chat rainbow "what are the colors of a rainbow"
sgpt --chat rainbow "inverse the list of your last answer"
sgpt --chat rainbow "translate your last answer in french"
```

你还可以使用提供的 `Dockerfile` 来构建自己的镜像:
```shell
docker build -t sgpt .
```

附加文档: [Azure 集成](https://github.com/TheR1D/shell_gpt/wiki/Azure), [Ollama 集成](https://github.com/TheR1D/shell_gpt/wiki/Ollama).
