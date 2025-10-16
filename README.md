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

### Simple Graph

1. **What I learned?** Built my first basic graph with LangGraph's core components. Learned that state is the object passed between nodes (here, a dictionary with one key graph_state). Nodes are Python functions that take in state and update it - node 1 appends "I am", node 2 appends "happy", node 3 appends "sad". Normal edges connect nodes in a fixed path (start → node 1). Conditional edges use logic to choose paths - here a function called decide_mood randomly picks node 2 or 3 with 50-50 odds.
Learned to build graphs using StateGraph class - add nodes, define edges with add_edge() for normal edges and add_conditional_edges() for conditional routing, then compile. The graph visualization shows dotted lines for conditional edges. Graphs use the "runnable protocol" with methods like invoke() which runs the whole graph synchronously, waiting for each step before moving to the next, and returns the final state after all nodes execute.
So we strted by understanding a simple graph: start → node 1 → (conditional) node 2 or 3 → end. Walked through defining state as a dictionary, creating three node functions that append text to graph_state, implementing decide_mood conditional edge with random logic, then assembling everything with StateGraph, adding nodes/edges, compiling, and visualizing. Ran the graph multiple times with invoke() starting from "Hi, this is Lance" and saw different outputs based on random routing.

2. **Code Tweaks?** Changed the input name from "Lance" to "Sumedha" to personalize it. Added an extra cell to run the graph multiple times and see different random outcomes.
Link: simple_graph.ipynb

### LangGraph Studio

1. **What I learned?** Learned how to work with LangGraph Studio and understand the project directory structure. Each module has its own folder with a studio directory inside. The studio folder contains: a requirements.txt file (for packages), Python scripts (defining the various graphs), and an .env file (for API keys). This entire directory can be loaded as a project in LangGraph Studio. Studio provides a visual interface to interact with and test the graphs we build - can see all graphs in the project and switch between them to play with different implementations.
it alsp showed how Studio displays the graphs visually.

2. **Code Tweaks?** Updated the whole `/studio` to use Mistral AI (free) instead of OpenAI.


### Chain with Messages and Tools

1. **What I learned?** Figured out how chat messages work as graph state and how the add_messages reducer keeps appending messages instead of overwriting them. Learned that chat models interact with messages - you can create a list of messages (with roles like AI/human and content) and pass them directly to a chat model to get an AI message response back with content and metadata. Started with messages basics - creating message lists with AI/human roles, passing to chat models, getting AI message responses with content and metadata.
Also learned how to bind tools to models so they know when to call functions. You define a function (like multiply), use llm.bind_tools(), and the model can produce tool calls instead of text responses. The tool call output contains arguments and the function name needed to run the tool. Then introduced tools - binding functions to LLMs with bind_tools(), invoking with questions like "what is 2 multiplied by 3", getting tool call outputs with arguments and function names.
Understood reducer functions - by default LangGraph overwrites state values, but with messages we want to append to preserve conversation history. The add_messages reducer does this automatically. LangGraph has a built-in MessagesState class that includes this reducer. Explained using messages as graph state, the need for add_messages reducer to append instead of overwrite, and the built-in MessagesState. Built a simple chain graph with messages as state, a single node (tool-calling LLM), and tested it with both natural language (got text response) and math questions (got tool calls). Built a graph with tool-calling LLM node, tested with "hello" (got natural language) and math question (got tool call).

2. **Code Tweaks?** Moved to Mistral AI, changed the tool test from "2 multiplied by 3" to "4 multiplied by 7" to use different numbers.
Link: chain.ipynb