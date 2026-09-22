#  AI Agent Lab — Course Fee Assistant

##  Project Overview

This project explores the evolution from a **Plain Chatbot** to a **Rule-Based Workflow** and finally to an **AI Agent**.

The project uses a college course-fee assistant as a private-data scenario. The same user requests are handled using three different approaches to understand their differences in flexibility, decision-making, private-data access, tool usage, and multi-step task handling.

The project demonstrates the core concept:

> **AI Agent = LLM + Tools + Loop**

---

##  Objective

The objective of this project is to understand how different approaches solve the same problem:

- **Plain Chatbot** — uses an LLM to generate responses.
- **Rule-Based Workflow** — follows predefined rules and conditions.
- **AI Agent** — combines an LLM with tools and an iterative loop to decide what actions are required to complete a task.

---

##  Scenario: College Course Fee Assistant

The assistant works with the following private course-fee data:

| Course Code | Course Fee |
|-------------|------------|
| CS101 | ₹12,000 |
| AI202 | ₹18,000 |
| DS303 | ₹15,000 |

Example questions used in the experiment include:

- What is the fee for AI202?
- What is the total fee for CS101 and AI202 after a 10% scholarship?
- Is DS303 more expensive than CS101, and by how much?
- Write a two-line welcome message for new AI students.

---

##  Three Approaches

### 1. Plain Chatbot

The Plain Chatbot uses an LLM to understand the user's request and generate a response.

It does not directly access the private course-fee dictionary used by the application. Therefore, when the user asks for specific course fees, the chatbot does not automatically know the stored values.

**Flow:**

```text
User
 ↓
LLM
 ↓
Response