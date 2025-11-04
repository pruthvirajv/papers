# QueryGenerationPrompts — README

This README documents the purpose, expected inputs/outputs, and usage guidance for the prompt files in this folder:

- employee_prompts.prompt  
- explorationplan_prompts.prompt  
- motor_prompt.prompt  
- rightofway_prompts.prompt

Each `.prompt` file contains templates intended for automated query / instruction generation for OntoLLM recommendation workflows. Use the templates as-is or adapt variables/placeholders to your pipeline.

---

## Common conventions
- Placeholders use curly braces or clear token markers like `{ENTITY}`, `{CONTEXT}`, `{GOAL}` — replace them before sending to the model.
- Keep system-level instructions (role/behavior) separate from user examples when integrating with a multi-turn system.
- Prefer short, focused prompts for extraction tasks and richer prompts for plan- or explanation-style outputs.
- Include examples in the prompt to improve few-shot performance where useful.

---

## employee_prompts.prompt
Purpose:
- Generate queries related to employees: attributes, roles, relationships, filtering, and entity linking.

Expected input:
- An employee entity or dataset context (e.g., name, ID, attributes).
- Objective such as "summarize", "search", "extract relations", or "generate filters".

Expected output:
- Structured queries, attribute lists, natural-language questions, or extraction templates.

Usage notes:
- Use when mapping employee metadata to knowledge-graph queries or creating QA prompts about staff.
- Include role and privacy constraints if output may expose PII.

Example snippet:
- Generate a KG query for: List all the employees in HR department

---

## explorationplan_prompts.prompt
Purpose:
- Create exploratory plans and stepwise strategies for information discovery given an ontology or dataset.

Expected input:
- A research goal, ontology/context description, and constraints (time, depth).

Expected output:
- Ordered plan steps, prioritized queries, recommended resources, and stopping criteria.

Usage notes:
- Use for multi-step data exploration, active learning workflows, or when orchestrating sequential LLM calls.
- Include success metrics or resource limits to guide plan brevity.

Example snippet:
-  Generate a KG query for: What is the exploration plan with control number 52 

---

## motor_prompt.prompt
Purpose:
- Generate prompts related to motor actions, control descriptions, or domain-specific motor attributes (e.g., for robotics, vehicles, or machinery).

Expected input:
- Motor specs or desired action descriptions (e.g., torque, speed, safety constraints).

Expected output:
- Structured action commands, verification checks, or natural-language summaries of motor behavior.

Usage notes:
- Validate units and safety constraints before executing any generated control commands.
- Limit to simulation or planning output; do not use for direct actuation without further validation.

Example snippet:
- Generate a KG query for: What is the location of motor ACME Y2000?

---

## rightofway_prompts.prompt
Purpose:
- Generate prompts about right-of-way rules, conflict resolution, and traffic-priority scenarios.

Expected input:
- Scenario description (agents, positions, signals) and jurisdiction or rule-set.

Expected output:
- Decision rules, prioritized agent list, recommended maneuvers or clarifying questions.

Usage notes:
- Specify jurisdiction or traffic code to avoid ambiguous advice.
- Use strictly for planning/simulation; do not substitute for legal or safety authority.

Example snippet:
- Generate a KG query for: Provide temporary ROWs of estern region

---

## Contribution & Maintenance
- Keep prompts modular and well-documented.  
- Add example inputs/outputs to each `.prompt` file for faster re-use.  
- Version prompts alongside any ontology or schema changes.

---

