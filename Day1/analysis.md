# Agentic AI: Foundations and Open-Source Practice — Day 1 Analysis

## 1. Chosen Private-Data Scenario

For this task, I chose a **college course-fee assistant** as the private-data scenario. The private data is a small course-fee list stored locally in the project:

- CS101 = Rs. 12,000
- AI202 = Rs. 18,000
- DS303 = Rs. 15,000

The same questions were tested using three approaches: a plain chatbot, a rule-based workflow, and an AI agent. The purpose was to observe how each approach handles private data, decision-making, tools, and multi-step tasks.

The test questions were:

1. What is the fee for AI202?
2. What is the total fee for CS101 and AI202 after a 10% scholarship?
3. Is DS303 more expensive than CS101, and by how much?
4. Write a two-line welcome message for new AI students.

A separate challenge question was also tested: **"I can pay Rs. 30,000. Which two courses can I take together within this budget?"**

---

## 2. Plain Chatbot

The plain chatbot uses an LLM through the Groq API. It does not use the local `COURSE_FEES` dictionary as a tool or provide that private data to the model. Its main job is to send the user's question to the LLM and return the model's response.

For the course-fee questions, this limitation was visible. When asked for the AI202 fee, the chatbot did not know the locally stored amount of Rs. 18,000 and asked for more information instead of retrieving the private fee. Similarly, it could not calculate the scholarship-adjusted total because it did not have access to the course-fee data.

For the general welcome-message question, the chatbot could answer normally because that question did not require private data. This shows that a plain chatbot can generate language using its LLM, but it does not automatically have access to application-specific private data stored in a Python dictionary.

The plain chatbot therefore mainly provides a response using an LLM alone. It does not have application tools, a tool-execution loop, or predefined business rules for the course fees. Its limitation in this scenario is the lack of direct private-data access and deterministic calculations.

---

## 3. Rule-Based Workflow

The rule-based workflow does not use an LLM. Instead, it uses predefined Python rules and directly accesses the local `COURSE_FEES` dictionary.

The workflow first extracts course codes such as CS101, AI202, and DS303 from the question. It then looks up their fees. If the question contains the word "total", it adds the identified fees and can apply a percentage scholarship. For example, CS101 and AI202 have a combined fee of Rs. 30,000, and a 10% scholarship reduces it to Rs. 27,000.

This approach successfully answered the first two course-fee questions because those questions matched the rules that were implemented. However, the workflow could not answer the question asking whether DS303 was more expensive than CS101 because no comparison rule had been implemented. It also rejected the welcome-message question because it was outside the workflow's fee-related rules.

The workflow demonstrates that predefined rules can provide reliable and predictable results for cases that have been explicitly programmed. However, its flexibility is limited because every new type of question requires another rule. Even a small change in wording or task structure can cause the workflow to fail if the corresponding condition has not been implemented.

---

## 4. AI Agent

The AI agent combines an LLM, tools, and a loop. The agent has access to two tools: `get_course_fee` and `calculator`.

The `get_course_fee` tool retrieves a course fee from the private `COURSE_FEES` dictionary. The `calculator` tool evaluates basic arithmetic expressions. The agent does not simply execute one fixed sequence. Instead, the LLM decides whether a tool is needed, selects a tool, receives the tool result, and then continues the interaction until it can produce a final answer.

For example, for the question about AI202, the agent selected `get_course_fee` and received `18000`. It then used that result to produce the final answer. For the comparison between DS303 and CS101, it called `get_course_fee` for both courses and then called `calculator` with `15000-12000`, obtaining `3000`.

The welcome-message question did not require a tool, so the agent produced a response directly. This demonstrates that an agent can combine tool use with ordinary language generation.

The budget challenge also demonstrated multi-step reasoning over the available course fees. The agent examined possible course combinations and identified combinations that fit within the Rs. 30,000 budget.

During testing with the Groq `openai/gpt-oss-20b` model, one tool-calling limitation was observed: for the scholarship question, the model sometimes generated the malformed tool name `calculator<|channel|>commentary`, which Groq rejected before the local Python tool could execute. Other calculator calls worked correctly. This is an observed limitation of the particular model/tool-calling interaction rather than a problem with the course-fee lookup tool itself.

Overall, the agent demonstrates the Unit 1 idea of **LLM + Tools + Loop**: the LLM decides what action is needed, tools provide access to application data or computation, tool results are returned to the LLM, and the process can continue until a final response is produced.

---

## 5. Comparison

| Basis for comparison | Plain chatbot | Rule-based workflow | AI agent |
|---|---|---|---|
| Flexibility | High for natural-language generation, but limited when private application data is required | Low to medium because supported cases must be explicitly programmed | High because the LLM can select different tools and handle different task structures |
| Decision-making | The LLM generates a response but has no application tools in this scenario | Decisions follow predefined Python conditions | The LLM decides which available tool to use and can perform multiple tool calls |
| Tool usage | No tools in this implementation | No LLM tools; uses direct Python rules and data access | Uses `get_course_fee` and `calculator` |
| Private-data access | No direct access to the local course-fee dictionary | Direct access to `COURSE_FEES` | Accesses private data through `get_course_fee` |
| Multi-step task handling | Limited in this implementation | Limited to the steps explicitly programmed | Can perform multiple tool calls and continue the agent loop |
| Automation | Generates responses automatically, but cannot retrieve the private fee list | Automates predefined fee operations | Automates tool selection, data retrieval, calculations, and final response generation |
| Reliability | Can avoid guessing, but cannot answer private-data questions without the data | Predictable for implemented cases; fails when no rule exists | Can handle more varied tasks, but tool-calling/model behavior can introduce failures |

---

## 6. Suitability Analysis

For this particular college course-fee scenario, the three approaches demonstrate different trade-offs. The plain chatbot is useful for general language questions such as writing a welcome message, but it cannot answer questions that depend on the private course-fee dictionary because the implementation does not give it access to that data.

The rule-based workflow is suitable when the questions are known in advance and the required operations are simple and predictable. It successfully retrieved fees and calculated the scholarship-adjusted total. However, the comparison question showed that the workflow needs a specific rule for every new type of operation.

The AI agent is suitable for the scenario when users may ask different combinations of fee-related questions. It can retrieve private data through a controlled tool, use a calculator for arithmetic, and perform multiple tool calls. Its flexibility comes from allowing the LLM to decide which available tool is appropriate instead of requiring one fixed rule for every question. At the same time, the observed malformed calculator tool-call error shows that agent reliability depends on the model and tool-calling implementation, so validation and error handling are important in a real application.

---

## 7. Conclusion

A **plain chatbot** is appropriate when the task mainly requires natural-language understanding or generation and does not depend on private application data or external actions. Examples include drafting messages, explaining concepts, or answering general questions.

A **rule-based workflow** is appropriate when the process is well-defined, the inputs and conditions are predictable, and consistent deterministic behavior is important. Examples include fixed calculations, form validation, and simple business rules.

An **AI agent** is appropriate when a task requires the system to decide what actions to take, use one or more tools, observe their results, and potentially perform multiple steps before completing the task. Examples include assistants that retrieve private records, perform calculations, search data, or coordinate several application operations.

The main difference demonstrated by this project is that a chatbot mainly produces a response, a rule-based workflow follows predefined steps and conditions, while an AI agent combines an LLM with tools and a loop so that it can select actions and use tool results as part of completing a task.
