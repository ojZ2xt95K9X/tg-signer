## Telegram每日自动签到/个人、群组、频道消息监控与自动回复

[English](./README_EN.md)

### 安装

```sh
pip install -U tg-signer
```

或者为了提升程序速度：

```sh
pip install "tg-signer[speedup]"
```

### 使用方法

```
Usage: tg-signer [OPTIONS] COMMAND [ARGS]...

  使用<子命令> --help查看使用说明

子命令别名:
  run_once -> run-once
  send_text -> send-text

Options:
  -l, --log-level [debug|info|warn|error]
                                  日志等级, `debug`, `info`, `warn`, `error`
                                  [default: info]
  --log-file PATH                 日志文件路径, 可以是相对路径  [default: tg-signer.log]
  -p, --proxy TEXT                代理地址, 例如: socks5://127.0.0.1:1080,
                                  会覆盖环境变量`TG_PROXY`的值  [env var: TG_PROXY]
  --session_dir PATH              存储TG Sessions的目录, 可以是相对路径  [default: .]
  -a, --account TEXT              自定义账号名称，对应session文件名为<account>.session  [env
                                  var: TG_ACCOUNT; default: my_account]
  -w, --workdir PATH              tg-signer工作目录，用于存储配置和签到记录等  [default:
                                  .signer]
  --help                          Show this message and exit.

Commands:
  list       列出已有配置
  login      登录账号（用于获取session）
  logout     登出账号并删除session文件
  monitor    配置和运行监控
  reconfig   重新配置
  run        根据任务配置运行签到
  run-once   运行一次签到任务，即使该签到任务今日已执行过
  send-text  发送一次消息, 请确保当前会话已经"见过"该`chat_id`
  version    Show version
```



例如:

```sh
tg-signer run
tg-signer run my_sign  # 不询问，直接运行'my_sign'任务
tg-signer run-once my_sign  # 直接运行一次'my_sign'任务
tg-signer send-text 8671234001 /test  # 向chat_id为'8671234001'的聊天发送'/test'文本
tg-signer send-text --delete-after 1 8671234001 /test  # 向chat_id为'8671234001'的聊天发送'/test'文本, 并在1秒后删除发送的消息
tg-signer monitor run  # 配置个人、群组、频道消息监控与自动回复
```

### 配置代理（如有需要）

`tg-signer`不读取系统代理，可以使用环境变量 `TG_PROXY`或命令参数`--proxy`进行配置

例如：

```sh
export TG_PROXY=socks5://127.0.0.1:7890
```

### 登录

```sh
tg-signer login
```

根据提示输入手机号码和验证码进行登录并获取最近的聊天列表，确保你想要签到的聊天在列表内。

### 发送一次消息

```sh
tg-signer send-text 8671234001 hello  # 向chat_id为'8671234001'的聊天发送'hello'文本
```

### 运行签到任务

```sh
tg-signer run
```

或预先执行任务名：

```sh
tg-signer run linuxdo
```

根据提示进行配置即可。

#### 示例：

```
开始配置任务<linuxdo>
第1个签到
1. Chat ID（登录时最近对话输出中的ID）: 10086
2. 签到文本（如 /sign）: /check_in
3. 等待N秒后删除签到消息（发送消息后等待进行删除, '0'表示立即删除, 不需要删除直接回车）, N: 5
继续配置签到？(y/N)：n
4. 每日签到时间（如 06:00:00）: 08:10:00
5. 签到时间误差随机秒数（默认为0）: 10
```

### 配置与运行监控

```sh
tg-signer monitor run my_monitor
```

根据提示进行配置。

#### 示例：

```
开始配置任务<my_monitor>
聊天chat id和用户user id均同时支持整数id和字符串username, username必须以@开头，如@neo

配置第1个监控项
1. Chat ID（登录时最近对话输出中的ID）: -4573702599
2. 匹配规则('exact', 'contains', 'regex'): contains
3. 规则值（不可为空）: kfc
4. 只匹配来自特定用户ID的消息（多个用逗号隔开, 匹配所有用户直接回车）: @neo
5. 默认发送文本: V Me 50
6. 从消息中提取发送文本的正则表达式:
7. 等待N秒后删除签到消息（发送消息后等待进行删除, '0'表示立即删除, 不需要删除直接回车）, N:
继续配置？(y/N)：y

配置第2个监控项
1. Chat ID（登录时最近对话输出中的ID）: -4573702599
2. 匹配规则('exact', 'contains', 'regex'): regex
3. 规则值（不可为空）: 参与关键词：「.*?」
4. 只匹配来自特定用户ID的消息（多个用逗号隔开, 匹配所有用户直接回车）: 6181244351
5. 默认发送文本:
6. 从消息中提取发送文本的正则表达式: 参与关键词：「(?P<keyword>(.*?))」\n
7. 等待N秒后删除签到消息（发送消息后等待进行删除, '0'表示立即删除, 不需要删除直接回车）, N: 5
继续配置？(y/N)：n
```

#### 示例解释：

1. 聊天`chat id`和用户`user id`均同时支持整数**id**和字符串**username**, username**必须以@开头** 如"neo"输入"@neo"，注意**username** 可能不存在，示例中`chat id`为-4573702599表示规则只对-4573702599对应的聊天有效。

2. 匹配规则，目前皆**忽略大小写**：

   1. `exact` 为精确匹配，消息必须精确等于该值

   2. `contains` 为包含匹配，如contains="kfc"，那么只要收到的消息中包含"kfc"如"I like MacDonalds rather than KfC"即匹配到（注意忽略了大小写）

   3. `regex` 为正则，参考  [Python正则表达式](https://docs.python.org/zh-cn/3/library/re.html) ，在消息中有**搜索到该正则即匹配**，示例中的 "参与关键词：「.*?」" 可以匹配消息： "新的抽奖已经创建...
      参与关键词：「我要抽奖」

      建议先私聊机器人"

   4. 可以只匹配来自特定用户的消息，如群管理员而不是随便什么人发布的抽奖消息

   5. 可以设置默认发布文本， 即只要匹配到消息即默认发送该文本

   6. 提取发布文本的正则，例如 "参与关键词：「(.*?)」\n" ，注意用括号`(...)` 捕获要提取的文本， 可以捕获第3点示例消息的关键词"我要抽奖"并自动发送



### 配置与数据存储位置

数据和配置默认保存在 `.signer` 目录中。然后运行 `tree .signer`，你将看到：

```
.signer
├── latest_chats.json  # 获取的最近对话
├── me.json  # 个人信息
└── signs
    └── linuxdo  # 签到任务名
        ├── config.json  # 签到配置
        └── sign_record.json  # 签到记录

3 directories, 4 files
```
