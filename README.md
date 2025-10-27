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
**Link:** simple_graph.ipynb

### LangGraph Studio

1. **What I learned?** Learned how to work with LangGraph Studio and understand the project directory structure. Each module has its own folder with a studio directory inside. The studio folder contains: a requirements.txt file (for packages), Python scripts (defining the various graphs), and an .env file (for API keys). This entire directory can be loaded as a project in LangGraph Studio. Studio provides a visual interface to interact with and test the graphs we build - can see all graphs in the project and switch between them to play with different implementations.
it alsp showed how Studio displays the graphs visually.

2. **Code Tweaks?** Updated the whole `/studio` to use Mistral AI (free) instead of OpenAI.


### Chain with Messages and Tools

1. **What I learned?** Figured out how chat messages work as graph state and how the add_messages reducer keeps appending messages instead of overwriting them. Learned that chat models interact with messages - you can create a list of messages (with roles like AI/human and content) and pass them directly to a chat model to get an AI message response back with content and metadata. Started with messages basics - creating message lists with AI/human roles, passing to chat models, getting AI message responses with content and metadata.
Also learned how to bind tools to models so they know when to call functions. You define a function (like multiply), use llm.bind_tools(), and the model can produce tool calls instead of text responses. The tool call output contains arguments and the function name needed to run the tool. Then introduced tools - binding functions to LLMs with bind_tools(), invoking with questions like "what is 2 multiplied by 3", getting tool call outputs with arguments and function names.
Understood reducer functions - by default LangGraph overwrites state values, but with messages we want to append to preserve conversation history. The add_messages reducer does this automatically. LangGraph has a built-in MessagesState class that includes this reducer. Explained using messages as graph state, the need for add_messages reducer to append instead of overwrite, and the built-in MessagesState. Built a simple chain graph with messages as state, a single node (tool-calling LLM), and tested it with both natural language (got text response) and math questions (got tool calls). Built a graph with tool-calling LLM node, tested with "hello" (got natural language) and math question (got tool call).

2. **Code Tweaks?** Moved to Mistral AI, changed the tool test from "2 multiplied by 3" to "4 multiplied by 7" to use different numbers.
**Link:** chain.ipynb


### Router

1. **What I learned?** Built a router that uses conditional edges to decide between calling a tool or responding directly. The model looks at the input and picks the right path - either runs the tool or just answers normally. This is a simple agent where the LLM directs control flow by choosing between two options based on input.
Learned about two key built-in LangGraph components: ToolNode (executes tool calls - just pass it your function as a list) and tools_condition (a prebuilt conditional edge that checks if the LLM output is a tool call). If it's a tool call, routes to the tools node; otherwise routes to end. The graph flow: input → chat model (bound with tools) → conditional edge checks output → either routes to tool node (executes function, returns tool message) or routes to end (direct natural language response).
Tested with two inputs: multiply question routes to tools node and executes the function, while "Hello World" gets direct response without routing to tools. Also saw how to run the router in Studio - opened the router.py script, verified it in langgraph.json, ran it in Studio interface. Studio visualizes tool calls nicely with formatted arguments and shows different routing paths clearly. Can see how direct responses go straight to end versus tool calls that route through the tools node.

2. **Code Tweaks?** Switched to Mistral AI (mistral-small-latest), changed the test question from "2 multiplied by 2" to "5 multiplied by 8", and added a friendly greeting test.
**Link:** router.ipynb


### Agent with ReAct Loop

1. **What I learned?** Built a proper agent that can chain multiple tool calls together - it calls a tool, sees the result, decides if it needs another tool, and keeps going until done. Pretty cool how it handles complex math step by step. Learned the ReAct architecture (Reason + Act) which is a popular generic agent pattern with three components: Act (let model call tools), Observe (pass tool output back to model), and Reason (let model reason about tool output and decide next step). The key difference from a router: instead of ending after a tool call, the tool message loops back to the model, allowing sequential tool calls until the model decides the problem is solved.
The modification from router to agent is simple - just add an edge from tools node back to the assistant node instead of going to end. This creates a loop that continues as long as the model makes tool calls (though in practice you'd add max iteration limits). Tested with a complex multi-step problem: add 3+4, multiply by 2, divide by 5. The agent made three sequential tool calls, reasoning between each: first got 7, then multiplied to get 14, then divided to get final answer 2.8.
Also learned about LangSmith tracing - set environment variables (LANGSMITH_API_KEY, LANGCHAIN_PROJECT name), then viewed also detailed traces in the LangSmith browser interface at smith.langchain.com. we Can see every step: initial assistant invocation, system prompts, function calls with payloads, tool node executions, token usage, latency which was Very useful for debugging alongside LangGraph Studio.

2. **Code Tweaks?** Switched to Mistral AI (mistral-small-latest model), changed system prompt to "solves math problems carefully, one step at a time", and tested with different numbers (6+9 then *3 then /5 instead of 3+4).
**Link:** agent.ipynb (also agent.py in studio folder)


### Agent Memory

1. **What I learned?** Learned how agents can actually remember past interactions using LangGraph’s memory checkpointer. Normally, each graph run starts fresh, so if you say “multiply that by 2” after adding some numbers before, it forgets what “that” means. The checkpointer fixes this by saving the graph’s state after every step, things like inputs, outputs, and what comes next as little snapshots called checkpoints, all grouped under a thread ID.
When we run the next query with the same thread ID, the agent picks up right where it left off it remembers the previous context (like knowing “7” came from 3+4). I also found it really cool that LangGraph Studio handles this automatically, using Postgres as its persistence layer, so you don’t even have to configure memory manually. Overall, it’s a simple but powerful way to make agents feel more aware and conversational.

2. **Code Tweaks?** Switched from OpenAI to Mistral AI, tweaked the system prompt to say "solving math problems step by step" instead of "performing arithmetic", and used different numbers (5+8 instead of 3+4) to test it out.
Link: agent_memory.ipynb


### Deployment with LangGraph Studio

1. **What I learned?** Learned how to actually deploy the agents we’ve been building. Understood how LangGraph’s API, Studio, and SDK all connect together. The API basically bundles your Python graph code and adds features like persistence and task queues. Studio (which we’ve already been using) is built on top of this same API, and it can run either locally or through LangGraph Cloud.
Also learned how deployment works using LangSmith we can connect our GitHub repo, deploy our graph, and instantly get a hosted URL with built-in monitoring, tracing, and documentation. The same graphs can be accessed through the SDK just by switching from the local Studio URL to the deployed cloud URL. Overall, it was a super clear walkthrough of how to move from local testing to a fully deployed, production-ready setup.

2. **Code Tweaks?** All setup uses Mistral AI instead of OpenAI. Changed the test input from “Multiply 3 by 2” to “Multiply 5 by 4” to keep things consistent with earlier notebooks. Also updated the Studio config files to work smoothly with the Mistral API. 
Link: deployment.ipynb


## Module 2

### Video 1: State Schema
In this video, I learned about Schema basically the structure and data types that our graph uses. So far, we were using TypedDict for this purpose. Then we were introduced to dataclass, which is quite similar but lets us access values using the “.” operator. One major issue with both dataclass and TypedDict is that type hints don’t get enforced at runtime, meaning we can still assign invalid values without getting an error. To fix this, Pydantic comes in really handy as it provides proper data validation. 

**Changes I Made:**

I created a graph that tracks a user’s preferred subscription plan (monthly vs yearly). Using Pydantic, the graph validates inputs and rejects invalid selections, showing clearly how runtime validation works while updating the state.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/state-schema.ipynb)

### Video 2: State Reducers
This video went deeper into reducers, which control how state updates happen for specific keys or channels. I learned that when two nodes try to update the same state at the same time, it throws a value error. To solve this, we can use state reducers, and Annotated helps in that. Sometimes we even need to make custom reducers for special cases like null. We also looked at message reducers, using the add_reducer method which helps us add, overwrite, or remove messages easily.

**Changes I Made:**

I tested built-in message reducers and created a custom score aggregator that sums quiz scores across nodes. This custom reducer ensures scores are combined correctly without errors, demonstrating practical use for state calculations across multiple nodes.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/state-reducers.ipynb)

### Video 3: Multiple Schemas
Here, I learned that most graphs use a single schema for all their input and output keys, but sometimes we need more control. There are cases where internal nodes send info that’s not relevant to the final output like internal communication and that’s where private state helps. It’s useful when we want to keep some internal logic hidden from the user. I also learned that while the StateGraph uses one schema for all communication, we can define explicit input and output schemas along with an internal schema for more complex designs.

**Changes I Made:**

I created a currency converter graph where private schemas handle intermediate calculations. Another graph collects user event preferences and processes them internally, then outputs only a summary message to the user. Both graphs demonstrate how private and public schemas interact.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/multiple-schemas.ipynb)


### Video 4: Trim and Filter Messages
In this video, I learned how to manage long conversations efficiently. When message threads get too long, they increase latency and token costs. To fix this, we can either delete older messages with `RemoveMessages`, or filter certain ones to pass only a subset to the model. Another trick is trimming, where we specify a token limit so only the latest relevant parts go to the LLM, making responses faster.

**Changes I Made:**

I built a customer feedback chatbot that keeps only the last three messages when interacting with users. Older messages are trimmed, ensuring the bot responds faster and uses fewer tokens while maintaining conversation continuity.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/trim-filter-messages.ipynb)


### Video 5: Chatbot with Summarizing Messages and Memory
This one was about making a chatbot that summarizes old messages once their count goes beyond six. This helps reduce context size and improves speed. I also learned about using a checkpointer for memory it saves graph state after every step, so even if the chat is interrupted, the bot can continue from where it left off. This persistence makes long-term conversations smoother and more practical.

**Changes I Made:**

I redesigned the chatbot as a virtual event assistant. After summarization, only the last two messages are retained, reducing LLM token usage. The bot maintains conversation flow while summarizing older interactions, and I tracked conversation threads in Langsmith to confirm persistence works.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/chatbot-summarization.ipynb)


### Video 6: Chatbot with External Database Memory
This video introduced persistent memory using external databases instead of temporary inmemory storage. I learned how to use SqliteSaver as a checkpointer, which writes conversation state to an actual database file on disk. This means the chatbot remembers everything even if you restart the notebook or reboot your computer, the conversation picks up exactly where you left off using the same thread ID.

**Changes I Made:**

Switched to Mistral AI and changed the example conversation to John Mayer topics (he's my fav). The chatbot now intelligently decides when conversations get too long and automatically summarizes them before continuing, creating a smooth flow for handling extended discussions with persistent memory.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-2/chatbot-external-memory.ipynb)


## Module 3
### Video 1: Streaming Responses with Interruption
In this lesson, I explored how LangGraph handles streaming outputs in different modes to create a real-time conversational experience. I learned to use .stream() with stream_mode='values' for full state streaming after each node and stream_mode='updates' to receive only incremental updates. I also experimented with .astream_events() to listen for real-time token generation via events like on_chat_model_stream. Finally, I tested remote streaming using the LangGraph API’s client.runs.stream() with the messages mode, confirming how both local and remote streaming can be implemented effectively.

**Changes:** Switched the entire implementation from OpenAI to Mistral AI (using mistral-large-latest model) and completely revamped all examples around John Mayer's music career. Instead of generic conversations, I created meaningful John Mayer-focused queries like discussing his guitar techniques, album evolution, live performances, collaborations, and gear setup. Each streaming method was tested with different aspects of his musical journey - from his signature playing style to his legendary guitar collection including vintage Stratocasters and PRS signatures. This created a cohesive theme while demonstrating how streaming works with both local graphs and the LangGraph API.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-3/streaming-interruption.ipynb)

### Video 2: Breakpoints
Here, I learned how to include a human-in-the-loop mechanism in LangGraph applications. This approach allows execution to pause and wait for human intervention or validation before continuing. It’s especially useful when the AI might be uncertain or when a human’s judgment is needed to verify a condition. I saw how interrupts can be triggered to inject user input dynamically, improving accuracy and oversight in AI workflows.

**Changes:** Migrated from OpenAI to Mistral AI and completely redesigned the tools around John Mayer's music career analytics. Instead of basic arithmetic functions (add, multiply, divide), I created custom music industry tools: calculate_album_sales (computes revenue from album units sold), calculate_tour_dates (determines total concert dates from monthly schedule), and calculate_setlist_duration (calculates show length in minutes). All examples were transformed into real John Mayer scenarios - calculating his tour schedule (8 concerts/month for 6 months), album revenue for 'Continuum' (500k units at $15 each), and setlist durations (18 songs at 5 minutes each). The breakpoint interrupts before tool execution, allowing human approval of these music calculations before proceeding.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-3/breakpoints.ipynb)

### Video 3: Edit State with Human Feedback
This lesson introduced how to pause, inspect, and modify the LangGraph’s state mid-execution, enabling dynamic debugging or feedback insertion. I learned to use interrupt_before=['node_name'] to pause execution before specific nodes, then get_state() to view and update_state() to modify data (like editing a message). I also practiced adding human feedback nodes that simulate user corrections in real-time, both locally and through the LangGraph API.

**Changes:** Built a John Mayer Career Analytics system where a music industry analyst can correct statistics mid-execution. Created tools for Grammy calculations (nominations + wins), collaboration counts across albums, and song catalog estimation. The flow pauses before the assistant node, allowing updates like changing "8 studio albums with 3 collaborations" to "9 albums with 4 collaborations" on the fly. Added a human_feedback node that intercepts queries (like correcting Grammy stats from estimate to actual: 19 nominations + 7 wins), applies the update using as_node="human_feedback", then continues with refined data.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-3/edit-state-human-feedback.ipynb)


### Video 4: Dynamic Breakpoints
In this lesson, I learned about dynamic breakpoints, a more flexible form of interrupts that can trigger based on runtime conditions. By raising a NodeInterrupt inside a node, I could create real-time checks where the graph pauses automatically if a certain condition (like a driver alert) is met. After pausing, I updated the graph state and resumed execution. I also visualized the entire process in LangSmith Studio to understand how interrupts appear in execution traces.

**Changes:** Built an album validation system for John Mayer's discography that dynamically interrupts when album names don't match his typical naming style. Created a three-step workflow (validate_album_format → check_album_length → process_album_data) where NodeInterrupt triggers if an album name exceeds 20 characters (since his actual albums like "Continuum", "Room for Squares", and "Battle Studies" are all concise). Tested with "The Complete John Mayer Guitar Collection" (42 chars) which raised the interrupt with a custom message explaining the issue, then updated state to "Continuum" to pass validation. This demonstrated conditional interrupts based on music industry naming conventions rather than fixed breakpoints.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-3/dynamic-breakpoints.ipynb)


### Video 5: Time Travel
This final lesson focused on LangGraph’s “time travel”—the ability to revisit, replay, and fork previous states. I learned to use get_state_history() to access the full execution history, replay past runs using stored configs, and modify (fork) them to explore alternative scenarios. This feature is especially powerful for debugging or testing different user inputs without restarting the whole graph. I also practiced replaying and forking remotely via the LangGraph API and visualized the version history in LangSmith Studio.

**Changes:** Created a John Mayer Career Statistics Time Travel system that allows rewinding and forking music industry calculations. Built tools for awards tracking (calculate_total_awards), chart performance (count_chart_positions), and revenue estimation (estimate_streaming_revenue). Started with calculating total awards (7 Grammys + 12 other = 19), then used get_state_history() to browse all checkpoint states. Demonstrated replaying from a previous checkpoint with the same data, and forking to explore alternative scenarios - changing from awards calculation to chart statistics (8 top 10 + 15 top 40 hits). Implemented both local forking (updating state with different John Mayer metrics) and API-based forking (modifying checkpoints remotely to test streaming revenue vs chart success scenarios). This showed how time travel enables "what-if" analysis for music career analytics without restarting the entire workflow.

File link: [click here](https://github.com/sumedhapaliwal/sumedhapaliwal_langgraph_mat496/blob/main/module-3/time-travel.ipynb)