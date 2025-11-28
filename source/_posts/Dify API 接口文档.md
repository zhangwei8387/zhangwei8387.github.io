# Dify API 接口文档

## 1. 发送对话消息 (Send Chat Message)

**接口地址**: `/chat-messages`  
**请求方式**: `POST`  
**描述**: 创建会话消息。

### Request Body

| 参数名 | 类型 | 必填 | 描述 |
| :--- | :--- | :--- | :--- |
| `query` | string | 是 | 用户输入/提问内容。 |
| `inputs` | object | 否 | 允许传入 App 定义的各变量值。`inputs` 参数包含了多组键值对（Key/Value pairs），每组的键对应一个特定变量，每组的值则是该变量的具体值。 默认 `{}`。 |
| `response_mode` | string | 是 | 响应模式：<br>- `streaming` 流式模式（推荐）。基于 SSE（Server-Sent Events）实现类似打字机输出方式的流式返回。<br>- `blocking` 阻塞模式，等待执行完毕后返回结果。（请求若流程较长可能会被中断）。由于 Cloudflare 限制，请求会在 100 秒超时无返回后中断。<br>**注**：Agent模式下不允许 `blocking`。 |
| `user` | string | 是 | 用户标识，用于定义终端用户的身份，方便检索、统计。由开发者定义规则，需保证用户标识在应用内唯一。服务 API 不会共享 WebApp 创建的对话。 |
| `conversation_id` | string | 否 | 会话 ID，需要基于之前的聊天记录继续对话，必须传之前消息的 `conversation_id`。 |
| `files` | array[object] | 否 | 文件列表，适用于传入文件结合文本理解并回答问题，仅当模型支持 Vision/Video 能力时可用。详见下方文件对象说明。 |
| `auto_generate_name` | bool | 否 | 自动生成标题，默认 `true`。若设置为 `false`，则可通过调用会话重命名接口并设置 `auto_generate` 为 `true` 实现异步生成标题。 |
| `workflow_id` | string | 否 | 工作流ID，用于指定特定版本，如果不提供则使用默认的已发布版本。 |
| `trace_id` | string | 否 | 链路追踪ID。适用于与业务系统已有的 trace 组件打通。如果未指定，系统会自动生成。支持以下三种方式传递（优先级递减）：<br>1. Header: `X-Trace-Id`<br>2. Query 参数: `trace_id`<br>3. Request Body: `trace_id` |

#### Files 参数对象说明

| 字段 | 类型 | 描述 |
| :--- | :--- | :--- |
| `type` | string | 支持类型：<br>- `document`: 'TXT', 'MD', 'MARKDOWN', 'MDX', 'PDF', 'HTML', 'XLSX', 'XLS', 'VTT', 'PROPERTIES', 'DOC', 'DOCX', 'CSV', 'EML', 'MSG', 'PPTX', 'PPT', 'XML', 'EPUB'<br>- `image`: 'JPG', 'JPEG', 'PNG', 'GIF', 'WEBP', 'SVG'<br>- `audio`: 'MP3', 'M4A', 'WAV', 'WEBM', 'MPGA'<br>- `video`: 'MP4', 'MOV', 'MPEG', 'WEBM'<br>- `custom`: 其他文件类型 |
| `transfer_method` | string | 传递方式：<br>- `remote_url`: 文件地址。<br>- `local_file`: 上传文件。 |
| `url` | string | 文件地址。（仅当传递方式为 `remote_url` 时）。 |
| `upload_file_id` | string | 上传文件 ID。（仅当传递方式为 `local_file` 时）。 |

### Example Request

```bash
curl -X POST 'http://10.20.6.34/v1/chat-messages' \
--header 'Authorization: Bearer {api_key}' \
--header 'Content-Type: application/json' \
--data-raw '{
  "inputs": {},
  "query": "What are the specs of the iPhone 13 Pro Max?",
  "response_mode": "streaming",
  "conversation_id": "",
  "user": "abc-123",
  "files": [
      {
        "type": "image",
      "transfer_method": "remote_url",
      "url": "https://cloud.dify.ai/logo/logo-site.png"
    }
  ]
}'
```

---

### Response

#### 1. Blocking 模式 (`response_mode` = `blocking`)

返回 `ChatCompletionResponse` 对象。Content-Type 为 `application/json`。

```json
{
    "event": "message",
    "task_id": "c3800678-a077-43df-a102-53f23ed20b88",
    "id": "9da23599-e713-473b-982c-4328d4f5c78a",
    "message_id": "9da23599-e713-473b-982c-4328d4f5c78a",
    "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2",
    "mode": "chat",
    "answer": "iPhone 13 Pro Max specs are listed here:...",
    "metadata": {
        "usage": {
            "prompt_tokens": 1033,
            "prompt_unit_price": "0.001",
            "prompt_price_unit": "0.001",
            "prompt_price": "0.0010330",
            "completion_tokens": 128,
            "completion_unit_price": "0.002",
            "completion_price_unit": "0.001",
            "completion_price": "0.0002560",
            "total_tokens": 1161,
            "total_price": "0.0012890",
            "currency": "USD",
            "latency": 0.7682376249867957
        },
        "retriever_resources": [
            {
                "position": 1,
                "dataset_id": "101b4c97-fc2e-463c-90b1-5261a4cdcafb",
                "dataset_name": "iPhone",
                "document_id": "8dd1ad74-0b5f-4175-b735-7d98bbbb4e00",
                "document_name": "iPhone List",
                "segment_id": "ed599c7f-2766-4294-9d1d-e5235a61270a",
                "score": 0.98457545,
                "content": "\"Model\",\"Release Date\",\"Display Size\",\"Resolution\",\"Processor\",\"RAM\",\"Storage\",\"Camera\",\"Battery\",\"Operating System\"\n\"iPhone 13 Pro Max\",\"September 24, 2021\",\"6.7 inch\",\"1284 x 2778\",\"Hexa-core (2x3.23 GHz Avalanche + 4x1.82 GHz Blizzard)\",\"6 GB\",\"128, 256, 512 GB, 1TB\",\"12 MP\",\"4352 mAh\",\"iOS 15\""
            }
        ]
    },
    "created_at": 1705407629
}
```

- `event`: 事件类型，固定为 `message`
- `task_id`: 任务 ID
- `id`: 唯一 ID
- `message_id`: 消息唯一 ID
- `conversation_id`: 会话 ID
- `mode`: App 模式，固定为 `chat`
- `answer`: 完整回复内容
- `metadata`: 元数据
- `usage`: 模型用量信息
- `retriever_resources`: 引用和归属分段列表
- `created_at`: 消息创建时间戳

#### 2. Streaming 模式 (`response_mode` = `streaming`)

返回 `ChunkChatCompletionResponse` 对象流式序列。Content-Type 为 `text/event-stream`。
每个流式块均为 `data:` 开头，块之间以 `\n\n` 分隔。

**模式说明 (基础助手 vs 智能助手)**:
- **基础助手 (Basic Assistant)**: 对应普通的对话应用。模型直接生成回复，主要返回 `message` 事件。
- **智能助手 (Agent Assistant)**: 对应 Agent 应用。模型具备推理和工具调用能力。除了返回文本内容的 `agent_message` 事件外，还会返回 `agent_thought` 事件，展示 Agent 的思考过程、工具调用（`tool`）、工具输入（`tool_input`）和工具执行结果（`observation`）。

**示例 1: 基础助手 (Basic Assistant)**
```
data: {"event": "message", "message_id": "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": " I", "created_at": 1679586595}
data: {"event": "message", "message_id": "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": "'m", "created_at": 1679586595}
data: {"event": "message", "message_id": "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": " glad", "created_at": 1679586595}
data: {"event": "message", "message_id": "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": " to", "created_at": 1679586595}
data: {"event": "message", "message_id" : "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": " meet", "created_at": 1679586595}
data: {"event": "message", "message_id" : "5ad4cb98-f0c7-4085-b384-88c403be6290", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "answer": " you", "created_at": 1679586595}
data: {"event": "message_end", "id": "5e52ce04-874b-4d27-9045-b3bc80def685", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "metadata": {"usage": {"prompt_tokens": 1033, "prompt_unit_price": "0.001", "prompt_price_unit": "0.001", "prompt_price": "0.0010330", "completion_tokens": 135, "completion_unit_price": "0.002", "completion_price_unit": "0.001", "completion_price": "0.0002700", "total_tokens": 1168, "total_price": "0.0013030", "currency": "USD", "latency": 1.381760165997548}, "retriever_resources": [{"position": 1, "dataset_id": "101b4c97-fc2e-463c-90b1-5261a4cdcafb", "dataset_name": "iPhone", "document_id": "8dd1ad74-0b5f-4175-b735-7d98bbbb4e00", "document_name": "iPhone List", "segment_id": "ed599c7f-2766-4294-9d1d-e5235a61270a", "score": 0.98457545, "content": "\"Model\",\"Release Date\",\"Display Size\",\"Resolution\",\"Processor\",\"RAM\",\"Storage\",\"Camera\",\"Battery\",\"Operating System\"\n\"iPhone 13 Pro Max\",\"September 24, 2021\",\"6.7 inch\",\"1284 x 2778\",\"Hexa-core (2x3.23 GHz Avalanche + 4x1.82 GHz Blizzard)\",\"6 GB\",\"128, 256, 512 GB, 1TB\",\"12 MP\",\"4352 mAh\",\"iOS 15\""}]}}
data: {"event": "tts_message", "conversation_id": "23dd85f3-1a41-4ea0-b7a9-062734ccfaf9", "message_id": "a8bdc41c-13b2-4c18-bfd9-054b9803038c", "created_at": 1721205487, "task_id": "3bf8a0bb-e73b-4690-9e66-4e429bad8ee7", "audio": "qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq"}
data: {"event": "tts_message_end", "conversation_id": "23dd85f3-1a41-4ea0-b7a9-062734ccfaf9", "message_id": "a8bdc41c-13b2-4c18-bfd9-054b9803038c", "created_at": 1721205487, "task_id": "3bf8a0bb-e73b-4690-9e66-4e429bad8ee7", "audio": ""}
```

**示例 2: 智能助手 (Agent Assistant)**
```
data: {"event": "agent_thought", "id": "8dcf3648-fbad-407a-85dd-73a6f43aeb9f", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "position": 1, "thought": "", "observation": "", "tool": "", "tool_input": "", "created_at": 1705639511, "message_files": [], "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_thought", "id": "8dcf3648-fbad-407a-85dd-73a6f43aeb9f", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "position": 1, "thought": "", "observation": "", "tool": "dalle3", "tool_input": "{\"dalle3\": {\"prompt\": \"cute Japanese anime girl with white hair, blue eyes, bunny girl suit\"}}", "created_at": 1705639511, "message_files": [], "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "message_file", "id": "d75b7a5c-ce5e-442e-ab1b-d6a5e5b557b0", "type": "image", "belongs_to": "assistant", "url": "http://127.0.0.1:5001/files/tools/d75b7a5c-ce5e-442e-ab1b-d6a5e5b557b0.png?timestamp=1705639526&nonce=70423256c60da73a9c96d1385ff78487&sign=7B5fKV9890YJuqchQvrABvW4AIupDvDvxGdu1EOJT94=", "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_thought", "id": "8dcf3648-fbad-407a-85dd-73a6f43aeb9f", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "position": 1, "thought": "", "observation": "image has been created and sent to user already, you should tell user to check it now.", "tool": "dalle3", "tool_input": "{\"dalle3\": {\"prompt\": \"cute Japanese anime girl with white hair, blue eyes, bunny girl suit\"}}", "created_at": 1705639511, "message_files": ["d75b7a5c-ce5e-442e-ab1b-d6a5e5b557b0"], "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_thought", "id": "67a99dc1-4f82-42d3-b354-18d4594840c8", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "position": 2, "thought": "", "observation": "", "tool": "", "tool_input": "", "created_at": 1705639511, "message_files": [], "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_message", "id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "answer": "I have created an image of a cute Japanese", "created_at": 1705639511, "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_message", "id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "answer": " anime girl with white hair and blue", "created_at": 1705639511, "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_message", "id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "answer": " eyes wearing a bunny girl" ,"created_at": 1705639511, "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_message", "id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "answer": " suit .", "created_at": 1705639511, "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "agent_thought", "id": "67a99dc1-4f82-42d3-b354-18d4594840c8", "task_id": "9cf1ddd7-f94b-459b-b942-b77b26c59e9b", "message_id": "1fb10045-55fd-4040-99e6-d048d07cbad3", "position": 2, "thought": "I have created an image of a cute Japanese anime girl with white hair and blue eyes wearing a bunny girl suit.", "observation": "", "tool": "", "tool_input": "", "created_at": 1705639511, "message_files": [], "conversation_id": "c216c595-2d89-438c-b33c-aae5ddddd142"}
data: {"event": "message_end", "id": "5e52ce04-874b-4d27-9045-b3bc80def685", "conversation_id": "45701982-8118-4bc5-8e9b-64562b4555f2", "metadata": {"usage": {"prompt_tokens": 1033, "prompt_unit_price": "0.001", "prompt_price_unit": "0.001", "prompt_price": "0.0010330", "completion_tokens": 135, "completion_unit_price": "0.002", "completion_price_unit": "0.001", "completion_price": "0.0002700", "total_tokens": 1168, "total_price": "0.0013030", "currency": "USD", "latency": 1.381760165997548}, "retriever_resources": [{"position": 1, "dataset_id": "101b4c97-fc2e-463c-90b1-5261a4cdcafb", "dataset_name": "iPhone", "document_id": "8dd1ad74-0b5f-4175-b735-7d98bbbb4e00", "document_name": "iPhone List", "segment_id": "ed599c7f-2766-4294-9d1d-e5235a61270a", "score": 0.98457545, "content": "\"Model\",\"Release Date\",\"Display Size\",\"Resolution\",\"Processor\",\"RAM\",\"Storage\",\"Camera\",\"Battery\",\"Operating System\"\n\"iPhone 13 Pro Max\",\"September 24, 2021\",\"6.7 inch\",\"1284 x 2778\",\"Hexa-core (2x3.23 GHz Avalanche + 4x1.82 GHz Blizzard)\",\"6 GB\",\"128, 256, 512 GB, 1TB\",\"12 MP\",\"4352 mAh\",\"iOS 15\""}]}}
data: {"event": "tts_message", "conversation_id": "23dd85f3-1a41-4ea0-b7a9-062734ccfaf9", "message_id": "a8bdc41c-13b2-4c18-bfd9-054b9803038c", "created_at": 1721205487, "task_id": "3bf8a0bb-e73b-4690-9e66-4e429bad8ee7", "audio": "qqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqqq"}
data: {"event": "tts_message_end", "conversation_id": "23dd85f3-1a41-4ea0-b7a9-062734ccfaf9", "message_id": "a8bdc41c-13b2-4c18-bfd9-054b9803038c", "created_at": 1721205487, "task_id": "3bf8a0bb-e73b-4690-9e66-4e429bad8ee7", "audio": ""}
```

**流式块事件类型 (event)**:

| Event | 描述 | 结构关键字段 |
| :--- | :--- | :--- |
| `message` | LLM 返回文本块事件（完整文本以分块输出）。 | `task_id`, `message_id`, `conversation_id`, `answer`, `created_at` |
| `agent_message` | Agent模式下返回文本块事件（仅Agent模式）。 | `task_id`, `message_id`, `conversation_id`, `answer`, `created_at` |
| `agent_thought` | Agent模式下思考步骤，涉及工具调用（仅Agent模式）。 | `id`, `task_id`, `message_id`, `position`, `thought`, `observation`, `tool`, `tool_input`, `created_at`, `message_files`, `file_id`, `conversation_id` |
| `message_file` | 文件事件，表示有新文件需要展示。 | `id`, `type` (image), `belongs_to` (assistant), `url`, `conversation_id` |
| `message_end` | 消息结束事件，代表流式返回结束。 | `task_id`, `message_id`, `conversation_id`, `metadata`, `usage`, `retriever_resources` |
| `tts_message` | TTS 音频流事件（开启自动播放时）。内容为 Base64 编码的 MP3。 | `task_id`, `message_id`, `audio`, `created_at` |
| `tts_message_end` | TTS 音频流结束事件。 | `task_id`, `message_id`, `audio` (空), `created_at` |
| `message_replace` | 消息内容替换事件（命中审查时触发）。 | `task_id`, `message_id`, `conversation_id`, `answer` (替换内容), `created_at` |
| `error` | 异常事件，收到后即结束。 | `task_id`, `message_id`, `status`, `code`, `message` |
| `ping` | 每 10s 一次的 ping 事件，保持连接存活。 | - |

---

### Errors

| HTTP Code | Error Code | 描述 |
| :--- | :--- | :--- |
| 404 | - | 对话不存在 |
| 400 | `invalid_param` | 传入参数异常 |
| 400 | `app_unavailable` | App 配置不可用 |
| 400 | `provider_not_initialize` | 无可用模型凭据配置 |
| 400 | `provider_quota_exceeded` | 模型调用额度不足 |
| 400 | `model_currently_not_support` | 当前模型不可用 |
| 400 | `workflow_not_found` | 指定的工作流版本未找到 |
| 400 | `draft_workflow_error` | 无法使用草稿工作流版本 |
| 400 | `workflow_id_format_error` | 工作流ID格式错误，需要UUID格式 |
| 400 | `completion_request_error` | 文本生成失败 |
| 500 | - | 服务内部异常 |
