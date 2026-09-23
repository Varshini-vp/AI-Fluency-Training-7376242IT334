Day 2 Analysis: Direct Prompting, Chain-of-Thought, and ReAct

1. Scenario

For this experiment, I used a course-fee and scholarship scenario. The
scenario contains both calculation questions that require careful
multi-step reasoning and a question that requires external information
from tools.

The main ReAct question was:

Which is cheaper: CS101 and AI202 with a 10% scholarship, or all three
courses with a 25% scholarship? By how much?

The course-fee tools provided the following values:

CS101 = ₹12,000

AI202 = ₹18,000

DS303 = ₹15,000

The agent had to retrieve these values before calculating the two
scholarship options.

For the Chain-of-Thought and self-consistency experiment, I used this
question:

A student takes three courses costing ₹12,000, ₹18,000 and ₹15,000.
She gets a 15% scholarship on the total and pays the rest in 4 equal
instalments. How much is each instalment?

The correct calculation is:

Total = ₹12,000 + ₹18,000 + ₹15,000 = ₹45,000

Amount after 15% scholarship = ₹45,000 × 0.85 = ₹38,250

Each instalment = ₹38,250 ÷ 4 = ₹9,562.50

2. Direct Prompting

Direct prompting asks the model to answer the question directly without
explicitly requesting a step-by-step solution. In my experiment, the
system prompt instructed the model to give only the final answer and not
explain its reasoning.

The model therefore receives the question, uses the information
available in its context, and immediately produces an answer. There is
no visible reasoning trace and there is no tool call in this approach.

For example, when asked the instalment question, the direct-prompting
output was:

Rs 9,562.50 per instalment.

This answer was correct because all the required numbers were already
included in the question.

Direct prompting is suitable for simple questions where the required
information is already available and the calculation or reasoning is not
complicated. However, it has an important limitation in this scenario.
It cannot independently retrieve the course fees used in the ReAct
question because the direct-prompting setup does not have tool access.
If a required fact is missing from the prompt and is not available from
the model's knowledge, direct prompting cannot fetch that fact.

Another limitation is that the answer does not show how the result was
obtained. This makes it difficult to inspect the intermediate
calculation from the response alone.

3. Chain-of-Thought Prompting

For Chain-of-Thought prompting, I used a prompt that explicitly asked
the model to solve the problem step by step, number the steps, show the
calculation, and finish with a final answer.

The model therefore receives the question and produces a sequence of
visible solution steps before giving the final result.

For the instalment question, the Chain-of-Thought output calculated the
total course cost, applied the 15% scholarship, divided the remaining
amount into four instalments, and reached:

Final Answer: ₹9,562.50 per instalment.

The Chain-of-Thought approach was correct for all three reasoning
questions tested in my comparison script:

The scholarship and instalment calculation --- ₹9,562.50 per
instalment.

The computer-sharing calculation --- 90 student sittings.

The height comparison --- Ravi was identified as the tallest and
Priya as the shortest.

Compared with direct prompting, Chain-of-Thought provides a deeper and
more inspectable solution because the intermediate steps are displayed
in the experiment.

However, Chain-of-Thought does not automatically provide external
information. It can reason over information supplied in the question,
but it cannot use the course-fee tool in my setup. Therefore, if the
exact course fees were not provided, Chain-of-Thought alone would not
solve the information-retrieval part of the ReAct scenario.

It can also require more tokens and processing than a short direct
answer because it generates additional reasoning steps.

4. ReAct Agent

ReAct stands for Reasoning and Acting. Instead of solving the complete
problem in one response, the agent alternates between reasoning about
what information it needs, taking an action through a tool, observing
the result, and continuing until it can produce a final answer.

In my experiment, the ReAct trace showed five tool-related steps:

get_course_fee({'course_code': 'CS101'}) → 12000

get_course_fee({'course_code': 'AI202'}) → 18000

get_course_fee({'course_code': 'DS303'}) → 15000

calculator({'expression': '(12000+18000)*0.9'}) → 27000.0

calculator({'expression': '(12000+18000+15000)*0.75'}) → 33750.0

The agent then compared the two results:

CS101 + AI202 with 10% scholarship = ₹27,000

All three courses with 25% scholarship = ₹33,750

Difference = ₹6,750

Therefore, the first option was cheaper by ₹6,750.

This experiment demonstrates the main advantage of ReAct in this
scenario. The agent did not need all the course fees to be written
directly in the question. It could call the course-fee tool, observe the
returned values, and then use a calculator tool.

The ReAct trace also makes the actions and observations visible in my
experiment. However, the trace is longer than a direct answer and
requires multiple tool calls. This can increase execution time and cost.
It can also fail if a tool returns incorrect information, becomes
unavailable, or if the agent chooses an inappropriate action.

5. Comparison Table

Basis for         Direct prompting  Chain-of-Thought   ReAct agent
comparison

Reasoning depth   Low for this      Higher; explicitly Higher; combines
experiment; gives works through      reasoning with
the answer        multiple steps.    repeated actions and
directly.                            observations.

Tool usage        No tool access in No tool access in  Uses tools to
this experiment.  this experiment.   retrieve course fees
and perform
calculations.

Reliability on    Can work for      Better suited to   Suitable when both
multi-step        simple            multi-step         multi-step reasoning
questions         calculations but  reasoning when all and external
gives less        required           information are
opportunity to    information is     required.
inspect           available.
intermediate
steps.

Transparency      Only the final    Intermediate       Tool actions and
answer is         solution steps are observations are
visible.          visible in this    visible in the ReAct
experiment.        trace.

Speed / cost      Generally fastest Usually slower and Can be slower and
because it        more               more costly because
produces a short  token-intensive    it may make several
response without  because it         tool calls and
tool calls.       produces multiple  reasoning steps.
reasoning steps.

6. Self-Consistency Observation

For the self-consistency experiment, I used the scholarship and
instalment question and ran the same Chain-of-Thought prompt five times.

At a non-zero temperature of 0.8, all five runs produced the correct
numerical answer of ₹9,562.50 per instalment. However, the wording of
the final answers varied. Because my program compared the extracted
final-answer strings exactly, only 2 of the 5 runs were counted as the
same exact answer string.

This shows an important limitation of a simple majority-vote
implementation: two answers can be numerically equivalent but have
different wording. For example, 9,562.5 rupees and
9,562.50 Rs per instalment represent the same result but are different
strings.

When I repeated the experiment with temperature set to 0, the five runs
produced the same final-answer form in the observed output, giving a
majority of 5 out of 5. The numerical answer was again correct:
₹9,562.50 per instalment.

Therefore, the experiment showed that the non-zero-temperature runs can
produce different phrasings even when the underlying answer is correct,
while temperature 0 produced a much more consistent output in this test.

7. Suitability Analysis

For the complete course-fee scenario, the ReAct approach is the most
suitable because the problem requires both reasoning and external
information. The agent needed the fees for CS101, AI202, and DS303
before it could calculate and compare the scholarship options. ReAct
allowed the agent to retrieve those values through tools and then use a
calculator.

Direct prompting would be suitable if all the required facts were
already provided and the user only needed a short answer. It is simple
and fast, but it does not provide tool access or a visible reasoning
process in this experiment.

Chain-of-Thought is useful when the information is already available but
the problem requires several reasoning or calculation steps. My three
Chain-of-Thought questions were answered correctly, and the displayed
steps made the calculations easier to inspect. However, Chain-of-Thought
by itself did not solve the external-information requirement of the
ReAct scenario.

The self-consistency experiment also showed that repeated sampling can
produce different answer wording at a non-zero temperature. The
underlying numerical result remained correct in my five runs, but
exact-string majority voting did not treat equivalent phrasings as the
same answer. At temperature 0, the observed outputs were much more
consistent.

For this particular scenario, these observations make ReAct useful for
the complete task, Chain-of-Thought useful for reasoning-heavy tasks
with known information, and direct prompting useful for straightforward
questions where a concise answer is sufficient.

8. Conclusion

The three approaches differ mainly in how they handle reasoning and
external information.

Direct prompting is appropriate for straightforward questions where the
necessary information is already available and a short response is
sufficient. It is simple and fast, but it provides no visible reasoning
in this experiment and has no tool access.

Chain-of-Thought prompting is appropriate for problems that require
multiple reasoning or calculation steps when the required information is
already available. Asking the model to work through the problem step by
step can make the solution easier to inspect and can help with
structured multi-step calculations. However, it does not automatically
provide missing external facts.

ReAct is appropriate when a problem requires both reasoning and
interaction with external tools or information sources. It can decide
what information is needed, call a tool, observe the result, and
continue reasoning until it can produce a final answer. The trade-off is
that tool use introduces additional steps, execution time, and possible
points of failure.

Overall, the experiment demonstrates that there is no single approach
for every problem. The appropriate method depends on whether the problem
is simple, reasoning-heavy, or requires external information and tool
interaction.