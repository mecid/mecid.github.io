---
title: Agentic coding in Xcode
layout: post
category: Meta
---

Apple has finally released Xcode 26.3, which now supports agentic coding. In this article, I’ll guide you through configuring Xcode 26.3 and utilizing the latest best practices when using agentic tools for building apps on Apple platforms.

{% include friends.html %}

I assume you’re familiar with agentic coding tools like Codex or Claude Code. If not, they’re computer programs that allow you to share access to your codebase and engage in conversations with it to discuss your codebase, implement features, review solutions, and more.

Xcode 26.3 fully supports agentic coding tools like Codex, Claude Code, or any other third-party providers. To activate these tools, you need an active subscription to one of them. You can do this by navigating to *Xcode -> Settings -> Intelligence -> OpenAI -> Codex*. A similar setup is available for Claud Code.

Xcode not only enables the use of agentic coding tools but also provides a Model Context Protocol (MCP). The MCP allows access to Xcode features within the agentic coding tools. For instance, a coding agent can run a preview, compare it with your design requirements, or access the latest Apple documentation.

Xcode 26.3 comes with bundled Codex and Claude Code instances which are a bit outdated now. But no worries, you can easily replace the bundled version of them with the recent one.

For instance, if you’ve already installed Codex on your working machine using *Brew* (which I highly recommend), you can create a symbolic link to the directory where Xcode stores bundled versions.

> ln -sf $(which codex) ~/Library/Developer/Xcode/CodingAssistant/Agents/Versions/26.3/codex

> ln -sf $(which claude) ~/Library/Developer/Xcode/CodingAssistant/Agents/Versions/26.3/claude

The same approach applies to the skills folder. Skills are reusable knowledge that you can share with your agentic coding tool. For instance, it could be a document on how to cook data flow in features or some best practices on building animations, and so on. These documents may not always be necessary in your workflow, but you might need to plug them in occasionally.

> ~/Library/Developer/Xcode/CodingAssistant/codex/skills

> ~/Library/Developer/Xcode/CodingAssistant/ClaudeAgentConfig/skills

You can create symbolic links to these folders to share your skills and have a single source of them. I highly encourage you to install skills by Antoine van der Lee for SwiftUI and Swift Concurrency. It might be a game changing for you.

Agentic coding in Xcode 26.3 isn’t just a new checkbox in Settings — it’s a shift in how we build apps on Apple platforms. When you connect tools like Codex or Claude Code directly to Xcode and unlock MCP, your editor stops being just an IDE and starts becoming a collaborative environment. Your agent can reason about your codebase, access documentation, run previews, and align implementation with design intent — all within the same workflow.
