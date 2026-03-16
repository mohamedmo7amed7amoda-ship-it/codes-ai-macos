# Codes — AI Coding Assistant for macOS

> A native macOS app that uses local AI (Ollama) to fix, create, and manage Swift files — built entirely in SwiftUI.

![Platform](https://img.shields.io/badge/platform-macOS%2014%2B-blue?style=flat-square)
![Swift](https://img.shields.io/badge/Swift-5.9-orange?style=flat-square)
![AI](https://img.shields.io/badge/AI-Ollama%20%7C%20qwen2.5--coder-teal?style=flat-square)
![License](https://img.shields.io/badge/license-All%20Rights%20Reserved-red?style=flat-square)

---

## What it does

Codes is a personal macOS coding assistant that talks to a **local Ollama model** — no cloud, no API keys, everything runs on your Mac.

- 🔧 **Fix Swift files** — paste errors, get complete corrected files back
- 💬 **Chat in Egyptian Arabic / English** — casual, context-aware conversation
- 📂 **Folder access** — reads your Xcode project files directly
- 🖥️ **Split-screen editor** — view AI-generated code side by side with your chat
- 🧠 **Deep Think mode** — multi-phase reasoning pipeline for complex tasks
- 🌐 **Web Search** — DuckDuckGo-powered research injected into AI context
- 🖼️ **Vision support** — attach screenshots of Xcode errors for analysis
- ↩️ **Undo / Redo** — full snapshot history for every AI file edit

---

## Screenshots

![Code View](screenshots/code-view.png)
![Chat View](screenshots/chat-view.png)

---

## Architecture

```
SwiftUI macOS App
├── 3-column NavigationSplitView
│   ├── Sidebar     — conversations, file tree, deep web search
│   ├── Chat        — streaming AI responses with code blocks
│   └── Detail      — split-screen code editor
│
├── AI Layer
│   ├── OllamaService     — streaming chat via local HTTP API
│   ├── ChatViewModel     — @MainActor state, undo/redo, file ops
│   └── KnowledgeRouter   — memory + dialect + emotion few-shot
│
└── Local Services
    ├── FolderAccessService  — non-sandboxed file I/O
    ├── FileCommandService   — parses ```create / ```edit blocks
    ├── LinguisticMemory     — learns from conversation history
    └── WebSearchService     — multi-engine deep search
```

---

## Code Sample — Streaming AI Response

```swift
// ChatViewModel.swift (excerpt)
func runOllama(for userInput: String, attachments: [Attachment] = []) async {

    var context: [ChatMessage] = [systemPrompt]
    context += EgyptianDialectDB.relevantExamples(for: userInput, count: 3)
    context += await LinguisticMemory.shared.fewShotMessages(for: userInput)

    if let path = selectedFile,
       let code = try? String(contentsOfFile: path, encoding: .utf8) {
        context.append(.system("OPEN FILE:\n```swift\n\(code)\n```"))
    }

    context += messages.suffix(6)
    context.append(.user(userInput, attachments: attachments))

    try await ollama.streamChat(
        messages: context,
        model: selectModel(for: userInput, hasImages: !attachments.isEmpty)
    ) { [weak self] accumulated in
        await MainActor.run {
            self?.replaceLastAssistant(accumulated)
        }
    }

    if let reply = messages.last(where: { $0.role == .assistant })?.content,
       reply.contains("```create") || reply.contains("```edit") {
        let (summary, _) = await FileCommandService.shared.handle(aiResponse: reply)
        FolderAccessService.shared.refreshFiles()
        statusMessage = "✅ \(summary)"
    }
}
```

---

## Tech Stack

| Layer | Technology |
|---|---|
| UI | SwiftUI 5, NavigationSplitView, async/await |
| AI | Ollama (local) — qwen2.5-coder:7b, llava vision |
| Storage | Actor-based JSON persistence |
| File I/O | Non-sandboxed, security-scoped bookmarks |
| Search | DuckDuckGo · Brave · Bing · Wikipedia |
| Language | Swift 5.9, macOS 14+ |

---

## Why local AI?

- **Privacy** — code never leaves your machine
- **Speed** — no network latency for token streaming
- **Cost** — zero API fees
- **Offline** — works without internet

---

## License

Copyright © 2026 **Mohamed Youssef El Sayed Hassaan**
Nationality: Egyptian · DOB: 22/07/1997

All Rights Reserved. This software and its source code are the exclusive property of Mohamed Youssef El Sayed Hassaan. No part of this software may be reproduced, distributed, modified, sublicensed, or used in any form without express written permission from the owner.

---

*Built by Mohamed Youssef El Sayed Hassaan*
