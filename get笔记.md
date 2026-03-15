Get笔记 - 个人笔记和知识库管理工具。

当用户提到以下意图时使用此技能：
「记一下」「存到笔记」「保存到Get笔记」「记录到Get笔记」
「保存这个链接」「保存这张图」「查我的笔记」「找一下笔记」
「加标签」「删标签」「删笔记」
「查知识库」「建知识库」「把笔记加到知识库」「从知识库移除」
「知识库里订阅了哪些博主」「博主发了什么内容」「直播总结」「直播原文」

支持：纯文本笔记、链接笔记（自动抓取网页内容并生成摘要）、图片笔记（OCR识别）、知识库管理（含博主订阅列表、直播总结）。

## 环境变量要求

需要配置以下环境变量（在 `~/.zshrc` 中 export）：
- `GETNOTE_API_KEY` (必填) — 格式：`gk_live_xxx`，从 https://www.biji.com/openapi 获取
- `GETNOTE_CLIENT_ID` (必填) — 格式：`cli_xxx`，同上获取
- `GETNOTE_OWNER_ID` (可选) — 飞书 Owner ID，用于权限控制

## 参数解析

用户输入的参数 `$ARGUMENTS`，格式：
- 无参数：进入交互模式，询问用户想做什么
- 文本内容：直接创建纯文本笔记
- URL链接：创建链接笔记
- `查询 关键词`：搜索笔记
- `知识库`：列出所有知识库
- `标签 笔记ID 标签名`：添加标签

## 执行步骤

全程自动化，不要询问确认。

### 0. 环境检查

先检查环境变量是否配置：
```bash
echo "API_KEY=${GETNOTE_API_KEY:+已配置} CLIENT_ID=${GETNOTE_CLIENT_ID:+已配置}"
```
如果缺少必要变量，提示用户前往 https://www.biji.com/openapi 创建应用获取凭证。

### API 基础信息

- **Base URL**: `https://openapi.biji.com`
- **认证头**:
  - `Authorization: $GETNOTE_API_KEY`
  - `X-Client-ID: $GETNOTE_CLIENT_ID`

所有 API 请求用 Bash 的 curl 执行，统一带认证头：
```bash
curl -s "https://openapi.biji.com/open/api/v1/..." \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID"
```

### 快速决策表

| 用户意图 | 接口 | 方法 | 关键点 |
|---------|------|------|--------|
| 「记一下」「保存笔记」 | /open/api/v1/resource/note/save | POST | 同步返回 |
| 「保存这个链接」 | /open/api/v1/resource/note/save | POST | note_type:"link" → **必须轮询** |
| 「保存这张图」 | 见图片笔记流程 | — | **4步流程，必须轮询** |
| 「查我的笔记」 | /open/api/v1/resource/note/list?since_id=0 | GET | since_id=0 起始 |
| 「看原文/转写内容」 | /open/api/v1/resource/note/detail?id={note_id} | GET | audio.original / web_page.content |
| 「加标签」 | /open/api/v1/resource/note/tags/add | POST | |
| 「删标签」 | /open/api/v1/resource/note/tags/delete | POST | system 类型不可删 |
| 「删笔记」 | /open/api/v1/resource/note/delete | POST | 移入回收站 |
| 「查知识库」 | /open/api/v1/resource/knowledge/list?page=1 | GET | 含统计数据 |
| 「建知识库」 | /open/api/v1/resource/knowledge/create | POST | 每天限50个 |
| 「笔记加入知识库」 | /open/api/v1/resource/knowledge/note/batch-add | POST | 每批最多20条 |
| 「从知识库移除」 | /open/api/v1/resource/knowledge/note/remove | POST | |
| 「查任务进度」 | /open/api/v1/resource/note/task/progress | POST | 链接/图片笔记轮询用 |
| 「订阅了哪些博主」 | /open/api/v1/resource/knowledge/bloggers | GET | 按topic_id查 |
| 「博主发了什么内容」 | /open/api/v1/resource/knowledge/blogger/contents | GET | 需要follow_id |
| 「博主内容原文」 | /open/api/v1/resource/knowledge/blogger/content/detail | GET | 需要post_id |
| 「有哪些已完成直播」 | /open/api/v1/resource/knowledge/lives | GET | 按topic_id查 |
| 「直播总结/原文」 | /open/api/v1/resource/knowledge/live/detail | GET | 需要live_id |

### 1. 创建纯文本笔记

```bash
curl -s -X POST "https://openapi.biji.com/open/api/v1/resource/note/save" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID" \
  -H "Content-Type: application/json" \
  -d '{"title":"笔记标题","content":"Markdown内容","note_type":"plain_text","tags":["标签1"]}'
```

纯文本笔记同步返回，立即完成。

### 2. 创建链接笔记（异步）

```bash
# 步骤1：提交任务
curl -s -X POST "https://openapi.biji.com/open/api/v1/resource/note/save" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID" \
  -H "Content-Type: application/json" \
  -d '{"note_type":"link","link_url":"https://...","title":"链接标题"}'
```

返回 task_id 后，立即告知用户「链接已保存，正在抓取原文和生成总结...」

然后 10-30秒间隔轮询：
```bash
curl -s -X POST "https://openapi.biji.com/open/api/v1/resource/note/task/progress" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID" \
  -H "Content-Type: application/json" \
  -d '{"task_id":"task_abc123xyz"}'
```

轮询直到 status=success 或 failed。成功后调详情接口展示结果。

若响应包含 `duplicate_count > 0` 且没有 task_id，说明链接已存在，直接告知用户。

### 3. 创建图片笔记（异步，4步流程）

1. 获取上传凭证：`GET /open/api/v1/resource/image/upload_token?mime_type=jpg`
2. 上传文件到 OSS（字段顺序必须：key → OSSAccessKeyId → policy → signature → callback → Content-Type → file）
3. 提交笔记：`POST /open/api/v1/resource/note/save` (note_type:"img_text", image_urls:[access_url])
4. 轮询任务进度直到完成

### 4. 查询笔记列表

```bash
curl -s "https://openapi.biji.com/open/api/v1/resource/note/list?since_id=0" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID"
```

返回 notes[], has_more, next_cursor, total（每次20条）。后续翻页用 next_cursor 作为 since_id。

笔记类型：plain_text / img_text / link / audio / meeting / local_audio 等。

### 5. 笔记详情

```bash
curl -s "https://openapi.biji.com/open/api/v1/resource/note/detail?id={note_id}" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID"
```

详情独有字段：audio.original、audio.play_url、web_page.content、web_page.url、attachments[]。

### 6. 标签管理

添加标签：
```bash
curl -s -X POST "https://openapi.biji.com/open/api/v1/resource/note/tags/add" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID" \
  -H "Content-Type: application/json" \
  -d '{"note_id":123456789,"tags":["工作","重要"]}'
```

删除标签：
```bash
curl -s -X POST "https://openapi.biji.com/open/api/v1/resource/note/tags/delete" \
  -H "Authorization: $GETNOTE_API_KEY" \
  -H "X-Client-ID: $GETNOTE_CLIENT_ID" \
  -H "Content-Type: application/json" \
  -d '{"note_id":123456789,"tag_id":"123"}'
```

system 类型标签不可删除。

### 7. 知识库管理

列表：`GET /open/api/v1/resource/knowledge/list?page=1`
创建：`POST /open/api/v1/resource/knowledge/create` body: `{"name":"名称","description":"描述"}`（每天限50个）
知识库笔记：`GET /open/api/v1/resource/knowledge/notes?topic_id=abc123&page=1`
添加笔记到知识库：`POST /open/api/v1/resource/knowledge/note/batch-add` body: `{"topic_id":"abc123","note_ids":[id1,id2]}`（每批最多20条）
移除笔记：`POST /open/api/v1/resource/knowledge/note/remove` body: `{"topic_id":"abc123","note_ids":[id1]}`

知识库选择逻辑：用户说「存到对应知识库」时，先获取知识库列表，根据笔记内容模糊匹配知识库名称。置信度高直接执行，低则列出候选让用户选。用户未提及知识库时不要擅自存入。

### 8. 博主订阅

博主列表：`GET /open/api/v1/resource/knowledge/bloggers?topic_id={alias_id}&page=1`
博主内容列表：`GET /open/api/v1/resource/knowledge/blogger/contents?topic_id={alias_id}&follow_id={follow_id}&page=1`
博主内容详情（含原文）：`GET /open/api/v1/resource/knowledge/blogger/content/detail?topic_id={alias_id}&post_id={post_id_alias}`

### 9. 直播管理

已完成直播列表：`GET /open/api/v1/resource/knowledge/lives?topic_id={alias_id}&page=1`
直播详情（总结+原文）：`GET /open/api/v1/resource/knowledge/live/detail?topic_id={alias_id}&live_id={live_id}`

## 安全规则

- 笔记数据属于用户隐私，不在群聊中主动展示笔记内容
- 若配置了 GETNOTE_OWNER_ID，检查 sender_id 是否匹配
- API 返回 error.reason: "not_member" 或错误码 10201 时，引导开通会员：https://www.biji.com/checkout?product_alias=6AydVpYeKl
- 创建笔记建议间隔 1 分钟以上，避免触发限流

## 错误处理

| 错误码 | 说明 | 处理方式 |
|--------|------|---------|
| 10001 | 鉴权失败 | 检查 API Key 和 Client ID |
| 10201 | 非会员 | 引导开通会员 |
| 20001 | 笔记不存在 | 确认笔记 ID 正确 |
| 42900 | 限流 | 降低频率 |
| 50000 | 系统错误 | 稍后重试 |

注意：note_id 是 64 位整数，响应 JSON 可能包含未转义的控制字符，用 python3 -c 'import json,sys; ...' 做容错解析。
