# Learnings and Tweaks

## Module 1

### Introduction

1. **What I learned?** Got an overview of what agents are and how they work. Learned that the module will cover generic agent architectures, common challenges developers face, and why building custom gents with domain-specific workflows is important for reliability. Understood that LangGraph models agent workflows as graphs and will teach core concepts like state, nodes, edges, tools, messages, looping (React architecture), and memory. Also learned we'll be using LangGraph Studio throughout for building and debugging

2. **Code Tweaks?** None

### Motivation

1. **What I learned?** Understood why we need LangGraph - a single LLM is limited without tools, external context, or multi-step workflows. Learned the difference between chains (fixed control flow set by developers, very reliable) and agents (LLM-defined control flow). Agents exist on a spectrum from low control (routers choosing between narrow options) to high control (fully autonomous agents). The key challenge: as you give LLMs more control, reliability typically drops. LangGraph aims to "bend the reliability curve" - maintaining reliability even with high LLM control.
Learned that LangGraph uses graphs with nodes (steps like tool calls or retrieval) and edges (connections between nodes), allowing developers to combine fixed steps with LLM control. LangGraph's core pillars are: persistence, streaming, human-in-the-loop, and advanced controllability. Also learned LangGraph comes with an IDE (Studio) for debugging and works well with LangChain integrations (though LangChain is optional).
Got the course roadmap: Module 1 covers foundations, router, and tool-calling agent. Module 2 adds memory/chatbots. Module 3 introduces human-in-the-loop. Module 4 combines everything into a complex research assistant.

2. **Code Tweaks?** None