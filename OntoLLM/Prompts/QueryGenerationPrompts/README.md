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

Example Usage:
- Generate a KG query for: List all the employees in HR department

---

## explorationplan_prompts.prompt
Purpose:
- Create exploratory plans and stepwise strategies for information discovery given an ontology or dataset.

Example Usage:
-  Generate a KG query for: What is the exploration plan with control number 52 

---

## motor_prompt.prompt
Purpose:
- Generate prompts related to motor actions, control descriptions, or domain-specific motor attributes (e.g., for robotics, vehicles, or machinery).

Example Usage:
- Generate a KG query for: What is the location of motor ACME Y2000?

---

## rightofway_prompts.prompt
Purpose:
- Generate prompts about right-of-way rules, conflict resolution, and traffic-priority scenarios.

Example Usage:
- Generate a KG query for: Provide temporary ROWs of estern region

---

## Contribution & Maintenance
- Keep prompts modular and well-documented.  
- Add example inputs/outputs to each `.prompt` file for faster re-use.  
- Version prompts alongside any ontology or schema changes.

---

