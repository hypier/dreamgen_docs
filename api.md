# DreamGen API

我们为我们的 Opus V1+ 模型提供 API。该 API 可用于 [部分高级订阅计划](https://dreamgen.com/pricing)。请注意，DreamGen API 严禁用于共享， strictly 仅限个人使用。

您可以通过任何语言的 HTTP 请求与 API 进行交互。一些 API 也与 OpenAI 的客户端库兼容。

## 认证

DreamGen API 使用 API 密钥进行认证。您可以在 [您的账户页面](https://dreamgen.com/account/api-keys) 创建和删除 API 密钥。

要对 API 请求进行认证，您需要通过 HTTP `Authorization` 头提交您的 API 密钥：

```
Authorization: Bearer YOUR_DREAMGEN_API_KEY

```

**请保密您的 API 密钥。** 不要与他人分享，也不要公开暴露。

## Chat API

此API目前没有文档。

## 文本 API

`POST https://dreamgen.com/api/v1/model/completion`

### 请求

```
type CompletionRequest = {
  modelId: 'opus-v1-sm' | 'opus-v1-lg' | 'opus-v1-xl';
  input: string;
  samplingParams: SamplingParams;
};

type SamplingParams = {
  kind: 'basic';
  presencePenalty?: number;
  frequencyPenalty?: number;
  repetitionPenalty?: number;
  temperature?: number;
  minP?: number;
  topP?: number;
  topK?: number;

  // 最大输出令牌数。
  maxTokens?: number;

  // 最大输入 + 输出令牌数。
  maxTotalTokens?: number;

  // 生成将停止的字符串数组。
  // 停止序列包含在输出中。
  stopSequences?: string[];

  // 如果设置为 true，模型将不会生成 EOS 令牌。
  // 生成将不会停止，直到满足以下条件之一：
  // - 达到 `maxTokens` 限制；或
  // - 遇到 `stop` 序列；或
  // - 满足其他停止条件
  ignoreEos?: boolean;

  // 仅适用于 Opus V1+。
  // 模型允许生成的角色集，例如
  // ["text", "user"]。
  allowedRoles?: string[];

  // 仅适用于 Opus V1+。
  // 如果设置为 true，模型将不会生成
  // EOM 令牌 (`<|im_end|>`).
  disallowMessageEnd?: boolean;

  // 仅适用于 Opus V1+。
  // 如果设置为 true，模型将不会生成
  // EOM 令牌 (`<|im_end|>`)，直到消息
  // 至少有 `minimumMessageContentTokens` 个令牌。
  minimumMessageContentTokens?: number;
};
```

### 响应

响应以 JSON-lines 格式流式传输，每行包含一个类型为 `CompletionResponse` 的 JSON 对象，如下所定义。

```
type CompletionResponse = CompletionOkResponse | CompletionErrResponse;

type CompletionOkResponse = {
  success: true,

  // 生成的输出。
  // 根据请求参数，可以是完整输出或增量输出。
  output: string,

  // 生成停止的原因。
  // - length: 达到 maxTokens
  // - stop: 达到 stopSequences
  finishReason?: 'length' | 'stop' | undefined,

  // 使用的令牌和积分数量。
  usage: CompletionUsage
};

type CompletionErrResponse = {
  message: string,
  success: false,
  status?: number | undefined
};

type CompletionUsageSchema = {
  inputTokens: number,
  outputTokens: number,
  completionCredits: number
};
```

## OpenAI 兼容的 API

我们提供多个 OpenAI 兼容的 API，包括：

- 聊天 API: `POST https://dreamgen.com/api/openai/v1/chat/completions`
- 文本 API: `POST https://dreamgen.com/api/openai/v1/completions`
- 模型 API: `POST https://dreamgen.com/api/openai/v1/models`

## OpenAI 聊天完成

注意：这是一个无状态的 API，消息内容不会被记录或存储。

`POST https://dreamgen.com/api/openai/v1/chat/completions`

这是与 OpenAI 的流式 [聊天完成规范](https://platform.openai.com/docs/api-reference/chat) 兼容的原生聊天 API 版本。

请查看这个 [Python Colab](https://colab.research.google.com/drive/1QBORGyG3BdVlXH3gBmUHtE3m9-u9GTNG?usp=sharing)，了解如何使用官方的 OpenAI Python 客户端库。

### 模型规格

许多 OpenAI 客户端仅支持 `system`、`assistant` 和 `user` 角色。为了支持 Opus 的 `text` 角色，这对于故事创作和角色扮演是必要的，我们在 `model` 请求字段中编码额外信息。

可能的 `model` 值为：

- `"opus-v1-{size}/{MODEL_SPEC}"`

其中 `{size}` 可以是 `sm` 或 `lg`，而 `{MODEL_SPEC}` 可以是 `text`、`assistant` 或具有以下模式的 JSON：

```
type ModelSpec = {
  assistant: {
    role: 'assistant' | 'text';
    name?: string;
    open?: boolean;
  };
  user?: {
    role: 'user' | 'text';
    name?: string;
  };
};
```

`assistant` 和 `user` 规格决定了 OpenAI 的 `assistant` 和 `user` 角色如何转换为 Opus 角色。

您还可以使用以下简写：

- `opus-v1-{size}/text`：表示 `{"assistant": {"role": "text", "open": true}}`
- `opus-v1-{size}/assistant`：表示 `{"assistant": {"role": "assistant", "open": false}}`

### 附加请求参数：

这些是 DreamGen 的 OpenAI API 支持的附加参数：

- `min_p`
- `top_k`
- `repetition_penalty`

使用 OpenAI 的 Python SDK 时，可以通过 `extra_body` 参数传递这些参数：

```
completion = client.chat.completions.create(
  model="opus-v1-lg/text",
  stream=True,
  max_tokens=200,
  messages=[...],
  temperature=0.5,
  frequency_penalty=0.1,
  presence_penalty=0.1,
  extra_body={
    "min_p": 0.05,
    "repetition_penalty": 1.1
  }
)
```

### 不支持（被忽略）的请求参数：

- `logit_bias`
- `logprobs`
- `top_logprobs`
- `n`
- `response_format`
- `seed`
- `stream`
- `tools`
- `tool_choice`
- `user`
- `function_call`
- `functions`

## OpenAI 文本补全

注意：这是一个无状态的 API，提示内容不会被记录或存储。

`POST https://dreamgen.com/api/openai/v1/completions`

这是与 OpenAI 的流式 [文本补全规范](https://platform.openai.com/docs/api-reference/completions) 兼容的原生文本 API 版本。

请查看这个 [Python Colab](https://colab.research.google.com/drive/1QBORGyG3BdVlXH3gBmUHtE3m9-u9GTNG?usp=sharing)，了解如何使用官方的 OpenAI Python 客户端库。

文本 `prompt` 应遵循 [Opus V1 模型指南](/models/v1.md) 中描述的 ChatML+Text 提示格式。

`model` 可以是以下之一：

- `opus-v1-{size}`
- `opus-v1-{size}/text` \-\- 模型仅允许使用 `text` 角色
- `opus-v1-{size}/assistant` \-\- 模型仅允许使用 `assistant` 角色

其中 `{size}` 可以是 `sm`、`lg` 或 `xl`。

### 额外请求参数：

这些是 DreamGen 的 OpenAI API 支持的额外参数：

- `min_p`
- `top_k`
- `repetition_penalty`

在使用 OpenAI 的 Python SDK 时，可以通过 `extra_body` 参数传递这些参数：

```
completion = client.completions.create(
  model="opus-v1-sm/text",
  stream=True,
  max_tokens=200,
  prompt=prompt,
  temperature=0.5,
  frequency_penalty=0.1,
  presence_penalty=0.1,
  extra_body={
    "min_p": 0.05,
    "repetition_penalty": 1.1
  }
)
```

### 不支持（被忽略）的请求参数：

- `best_of`
- `echo`
- `logit_bias`
- `logprobs`
- `n`
- `seed`
- `stream`
- `suffix`
- `user`

## OpenAI 列出模型

`GET https://dreamgen.com/api/openai/v1/models/list`

此 API 与 OpenAI 的 [列出模型规范](https://platform.openai.com/docs/api-reference/models/list) 兼容。