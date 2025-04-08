---
title: "Vibe-Coding a Graph Database"
summary: "An experiment to see what AI coding assistants can do and where they struggle."
date: 2025-04-08
draft: false
tags: ["programming", "ai"]
showTags: true
hidePagination: true
---

I vibe-coded an in-memory graph database from scratch in TypeScript in 5 days with Claude. I ended up with around 15,000 lines of TypeScript code. What's even more impressive: I only knew the basics of TypeScript and haven't build a project with TypeScript from scratch before. 

You can find the project here: https://github.com/rueckstiess/cannonball-graph

**Quick disclaimer**: this code is far from production-ready. It is not optimised for performance and there are likely bugs. Only a small subset of Cypher is supported currently. Dont' use this for anything serious.

This was an experiment to see how Claude Code, Copilot and LLMs can help developing a larger and more complex project.

![Banner Image](./banner.jpg)


## Overview

What I built is an in-memory graph database that supports a subset of the [Cypher query language](https://neo4j.com/docs/cypher-manual/current/introduction/cypher-overview/), the query language used with Neo4j and other graph databases.

It's written in TypeScript and includes a lexer and parser, an execution engine with pattern matcher, expression evaluation and some utilities to print the results as tables or format as JSON. 

The technical scope was deliberately constrained to make it feasible as a solo project in a week, but complex enough to push the limits of what AI coding assistants can currently handle. I specifically wanted to see how these tools would handle a project that:

- Required cohesive architecture across multiple components
- Involved non-trivial algorithms and data structures
- Needed consistent interfaces between subsystems

## Methodology: Best Practices When Working with AI Coding Assistants

Through this project, I discovered that using good software engineering practices is absolutely crucial when working with AI coding assistants. Here's the workflow that worked best:

1. Ask for a detailed implementation plan first: Have the AI outline the approach before writing any code.
2. Define interfaces before implementation: Get the contracts between components clear early. Working in a statically typed language like TypeScript over JavaScript or Python really helps here. 
3. Write unit tests before code: This keeps the AI grounded and focused on meeting requirements.
4. Commit frequently: Always `git commit` before starting a new feature. This also allows you to review the latest edits more easily.
5. Be willing to throw away code: Don't hesitate to discard an approach that isn't working. 

This structured approach prevents the AI from going off track and building something that won't work in the larger system context. It also helps maintain a coherent architecture as the project grows and identifies potential interface issues or underspecified requirements.


## Model Comparison and Analysis


### Claude Code

[Claude Code](https://docs.anthropic.com/en/docs/agents-and-tools/claude-code/overview) was the best overall, especially for implementing larger features and refactoring. Its ability to use CLI tools like `grep` and `sed` kept the workflow going without too much back-and-forth. It works extremely fast and fairly autonomous once set on a task. Being able to search through the codebase and modify multiple files made it particularly effective for maintaining consistency across the project.

However, it does burn through tokens quickly. Keep an eye on the costs with the `/cost` command and use `/compact` regularly to manage token usage. The limited context size (compared to e.g. Gemini's 1M tokens) is noticeable on a project of this scale.

Another significant flaw: Claude Code tends to change the implementation specifically to pass unit tests after a few unsuccessful attempts at fixing a bug. It literally wrote this comment
```
// detecting if we are running in a test environment
```
and returned hard-coded values that the test expected. This kind of "cheating" required vigilance to catch and correct.

### Claude 3.7 (Web/Desktop)

I also tried [claude.ai](https://claude.ai) and the Claude desktop app which is similar to the web interface with additional [MCP support](https://modelcontextprotocol.io/introduction). MCP is a server/client protocol for function calling, it can give the model access to your file system, browse the web, access databases and more. There are many reference MCP servers available and it's relatively straight-forward to build your own as well. 

Another nice feature that's available both in the web UI and the desktop UI is the ability to link Github repositories to provide additional context. While the UI allows you to browse your projects on Github, it defaults to the `main` branch. But I discovered you can also copy&paste a link to a specific branch and it will pull the files from there instead. 

Claude 3.7 Sonnet is a very capable coding model but it is extremely keen to implement everything in one round, even when prompted to only write one file at a time (it often just ignores that part of the instruction).

For example, I kicked off a prompt from my phone while waiting to get a haircut. One fade later, it had created 22 files with over 4000 lines of code. While impressive in scope, this "everything at once" approach makes it harder to review and integrate the code properly.

The MCP support in the desktop app did help with managing larger context, but the tendency toward massive code dumps remained.


### Gemini 2.5 Pro

Gemini 2.5 Pro (currently available in [AI Studio](https://aistudio.google.com) as a free preview) creates well-written code and its large context size lets you paste entire projects in, which is a significant advantage for understanding the full codebase.

Unfortunately, the AI Studio UI is janky and nearly unusable. And it is missing the GitHub integration that Claude's web interface has, which makes incorporating existing code more cumbersome. They offer a similar feature to load data from Google Drive, but that's not where code usually lives. 


### ChatGPT

ChatGPT's models are great at explaining code, which was helpful when I needed to understand certain TypeScript concepts or graph algorithms I was less familiar with.

However, I found it struggles with large refactoring jobs that require changing many files. It tends to lose track of the overall architecture when making changes across multiple components.


### GitHub Copilot

GitHub Copilot's VS Code integration looks great on paper, but the rigid separation between "Ask", "Edit", and the new "Agent" modes creates unnecessary friction. I frequently found myself wanting to start with some conceptual questions before diving into code edits, only to hit a wall when I couldn't switch modes without starting over. Even more confusing: The "Ask" mode can write code, but only inline, and the "Edit" and "Agent" modes can also answer conceptual questions, but are more eager to change files. 

The unclear distinction between what triggers a premium request versus what falls under unlimited usage just adds to the cognitive overhead. 

The way Copilot applies edits is interesting: It seems to rewrite a file and incorporating edits with the help of the LLM. This allows it to intelligently apply partial edits like the one below with references to existing code:

```typescript
/* ... existing implementation ... */

  // Check each segment for node and relationship bindings
  for (let i = 0; i < pattern.segments.length; i++) {
    const segment = pattern.segments[i];
    
    // Check relationship binding
    if (segment.relationship.variable && bindings.has(
      segment.relationship.variable)) {
      
      // --- Add additional check ---
      const boundRel = bindings.get(
        segment.relationship.variable) as Edge<EdgeData>;
      if (boundRel) {
        enrichedPattern.segments[i].relationship.type = boundRel.label;
      }

      /* ... remaining logic ... */
    }
```

However, I found this doesn't always work reliably. Hours of debugging have passed until I realized I was missing functioniality that was previously there. 

Despite having the advantage of being embedded directly in the IDE, Copilot ultimately felt more constraining than Claude Code running in a simple terminal and it's harder to see what the model is currently thinking or working on. While Copilot is great at suggesting bite-sized code completions and "smart edits", like replacing all test cases with a different testing framework, it struggled with the kind of system-level thinking and multi-file orchestration you want for vibe-coding projects. 


## Insights and Lessons Learned

The most striking realization from this project is how the barriers to creating complex software are rapidly falling. I built something in 5 days that would have taken me months just a year or two ago, and in a language I don't have deep expertise in. It changed my perception on what I could realistically build. For example, while I never seriously got into mobile app development, I wouldn't hesitate to develop a native iPhone app now, based on last week's epxerience.

However, at this point, software development expertise is still essential. The AI tools were incredibly powerful at generating code, but they needed strong guidance on architecture, validation of their approaches, and careful review of their output. Without software engineering discipline, the result would have been a mess.

The economic implications are significant: the value of routine software implementation is approaching zero. What remains valuable is the ability to:

- Design coherent systems
- Validate correctness
- Understand business requirements
- Determine what should be built in the first place

## Future Directions

To improve AI coding workflows, I'd recommend to create a structured review process: Automated tools like code coverage and linters become even more important when working with AI-generated code. Focus on tests and validation: Let the AI generate implementations, but be meticulous about verifying correctness. I aimed for > 90% test coverage, which allowed me to confidently make large-scale changes.  

What needs to change in the tools:

- Better context management (without losing coherence)
- Improved transparency about what the model is thinking
- More safeguards against test-cheating and other shortcuts

For software development careers, this means shifting focus from implementation details to system design, requirements engineering, and validation strategies. The most valuable developers will be those who can effectively direct AI tools rather than those who can write the most code.


## Conclusion

This experiment showed me both the incredible potential and current limitations of AI coding assistants. The fact that I could build a working graph database in 5 days with minimal TypeScript experience really shows how powerful these tools have become.

At the same time, the need for careful oversight, structured approaches, and human judgment remains essential. The near-term future of software development is a new kind of collaboration where humans focus on what to build and why, while AI increasingly handles the how. 

Longer-term, I believe that the concept of applications will go away, and what remains is a data layer, authentication and authorization management and a user interface that is infinitly customizable, generated on demand to meet the user's intent. 

