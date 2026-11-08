---
name: justincourse-courses
description: 通过 justincourse MCP 服务读取用户在 justincourse.com 有权限的课程内容、搜索课程、回答课程相关问题。当用户提到 justincourse、Justin 的课程、课程里某个课时或知识点，或想查「我买的 / 我有权限的课程」时使用。
---

# JustinCourse 课程助手使用流程

本 skill 只描述使用流程。MCP 服务 `justincourse`（`https://justincourse.com/api/mcp`）提供的工具名称、参数与返回结构，一律以服务端 `tools/list` 为准，这里不复述。

## 连接与授权

首次使用时，客户端会打开浏览器，让你登录 justincourse.com 并授权。授权随时可以撤销：账户页「已授权应用」（`/authorized-apps`）。服务全部只读，且只能读到你本来就有权限的内容。

## 流程

1. **先定位，再取内容。** 用 `search_content` 搜索，或用 `list_my_courses` / `get_course_outline` 查看课程和目录，确定是哪门课、哪个课时。
2. **再用 `get_lesson` 取课时内容。** 正文和字幕较长时会分页，用返回的 `nextCursor` 继续取。只取回答问题需要的部分，不要一次性把整门课翻完。
3. **回答时引用来源。** 写明课时标题，并附该课时的链接（用工具结果里返回的页面 URL，不要自己拼）。
4. **遇到锁定内容，不要编造。** 工具返回「锁定」时，只依据返回的标题、简介等已有信息作答；说明需要哪个套餐才能查看，并给出定价页链接 https://justincourse.com/pricing 。没有取到的内容不要猜、不要凭印象补全。
5. **默认用中文回答。** 用户用其他语言提问时，跟随用户的语言。
6. **需要打开页面时。** 在 Claude Code 下可以用 `open <url>` 在浏览器里打开课时或定价页。
