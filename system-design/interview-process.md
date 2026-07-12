# System Design Interview Process

This is the reusable process for guided practice and Friday interview simulations.

## 1. Clarify the problem

Confirm the core user journey and system boundary.

## 2. Gather requirements

Ask only questions that can change the design, such as:

- expected load and whether it is constant or bursty;
- latency and availability expectations;
- delivery or consistency guarantees;
- multi-tenancy;
- scheduling, audit and compliance requirements;
- functional and non-functional scope.

During early sessions, the coach explains why each question matters.

## 3. Summarize

Use a short transition:

> Let me summarize my understanding before I proceed.

Repeat only the design-shaping requirements, not the full interviewer response.

## 4. State assumptions

State only assumptions that materially affect architecture.

## 5. Explain the approach

Example:

> I will identify the major responsibilities first, then build the high-level flow and deep-dive reliability, scalability and operations.

## 6. Identify responsibilities

Define who owns what before selecting products.

## 7. Build the high-level architecture

Show the main request/data path and asynchronous boundaries.

## 8. Deep dives

Use concepts already covered in the curriculum. Park uncovered concepts.

## 9. Trade-offs

Explain the selected option, an alternative, and why the selected choice fits the requirements.

## 10. End-to-end summary

Conclude with a concise request flow and the most important guarantees.
