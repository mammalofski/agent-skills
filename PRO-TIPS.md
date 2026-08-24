## PRO Tips


> **YOU** are the engineer and architect; the agent just implements what you architect.

- **NEVER ask the agent to just "do something."** ALWAYS ask the agent to plan first, review the plan, then execute after approval.

  - Unless the task is simple and you already have the plan, in which case, write the plan yourself, step by step.

  - **My current workflow:** (for medium-large tasks)
    - For small-medium tasks:   
        - No matter the size, I always tell the agent to understand the task and ask clarifying questions, read code and context, plan properly, execute thoroughly, verify, and close cleanly. To do so efficiently I use [quick-task](./skills/qt) and [quick-debug](./skills/qd) skills which does all the workflow for me. Just replace your usual day-to-day tasks with `/qt <task description>` or `/qd <bug description>`
    - For medium-large tasks
        - (optional but recommended) Prepare a well-crafted, feature and specs document. Have your initial requirements ready, then do a brainstorming session with the agent and have it to ask all the questions it needs from me to map out all the possible gray areas (bonus [grill-me](https://www.aihero.dev/skills-grill-me) or [grill-with-docs](https://www.aihero.dev/grill-with-docs) skills) if you already have the initial docs ready.
        - I first select plan mode in my agent, (and select the most powerful model I have i.e. Opus 5 xhigh, optionally use the [deep-plan](./skills/deep-plan) skill and) ask the agent to analyze, think hard, look at everything, make a good understanding, and make a perfect plan in `.planning/navoid-plans/<slug>/plan.md` with as much as detail as possible and an executive summary. I review the executive summary, ask for adjustments if needed, and then approve it.
        - (optional) Then have another agent revise the plan and improve.
        - have the third agent implement with [deep-execute](./skills/deep-execute) (can be done with a weaker model if planned properly i.e. Sonnet 5).
        - Finally a fourth agent write E2E tests and a QAT testing guide for me and I manually test it throughly. Works 99% of the time with no issues.
        - Finally if required, I ask the agent to fix anything I find in the same session or new sessions with [quick-task](./skills/qt) and [quick-debug](./skills/qd). (which if the specs and planning are done properly, it rarely happens)
    - For extra large tasks (like 0 to 1 implementations)
        - I use [GSD](https://github.com/open-gsd/gsd-core) framework. Very expensive and verbose, but effective.


- **NEVER let the agent decide the architecture.** ALWAYS plan the architecture yourself and let the agent implement the details.

  - If you don't have the architecture or plan yet, exploit the agent's internet access to research and brainstorm. Make the plan WITH it, don't let it make the plan FOR you.
- Always have a project AGENTS.md (or CLAUDE.md or whatever).

  - Always keep it updated; ask your agent to initialize it and update it after any major change. (i.e. have the instruction to update AGENTS.md at the end it)
  - Also have your general AI agent guidelines that you want your agent to follow in there.

    - good claude.md: https://github.com/forrestchang/andrej-karpathy-skills/blob/main/CLAUDE.md
  - <details>
    <summary><b>My current guidelines at the end of AGENTS.md (under <code>## Very important AI Agent instructions</code>)</b></summary>

    - Always understand the code before making any changes. Make sure to understand the code and the architecture of the project before making any changes; this will help you make better changes and avoid mistakes.
    - Whatever task you have, whatever change you make, make sure to have a holistic view of the project. Think about all the different changes you need to make, even if they were not mentioned. I.e., if a change requires a DB change, you need to create the Alembic (or any other) migration; if it requires an API change, you need to create the Pydantic schemas; if it requires new env vars, you need to update the ocp-secret-mapping.yaml and vault secrets; if it requires new dependencies, you need to update the requirements.txt file, etc.
    - Always plan ahead. Think about all the different changes you need to make, and then execute them one by one, making sure to test your changes locally before committing.
    - Always add an extra 2 steps at the end of all of your workflows: 1. In the end, act as a smart critic, review your changes and your work, and add feedback on what is missed, what to improve, etc. 2. Then go back and adjust your changes based on the feedback you gave yourself. (Optional 3: repeat until you are confident your work is perfect and the result is the best it can be.)
    - If any documentation like AGENTS.md, README.md, docs/ or any other documentation needs to be updated based on the changes you made, make sure to update it as well and keep it up to date.
    - Make use of delegating tasks to sub-agents if you need to save your context (let them do the work), improve performance (by delegating smaller, more accurate tasks to sub-agents), and make things parallel if possible.
    - **when reporting information to me, be extremely concise, sacrifice grammar for concision.**m Always Write in ASD-STE100 to avoid ambiguity.

    </details>
- **For multi-feature applications**, use a spec-driven approach (OpenSpec, SpekKit, or GSD - my personal fav) to define and track features, structure, and behavior before implementation.

  - Define top-down: break the application into features, and features into TODOs.
  - Verify feature-level specs.
  - ALWAYS ask the agent to verify and optimize the TODO lists based on your goals. You'll be surprised!
- **For repetitive tasks**, ALWAYS build custom skills and agents and/or plugins that employ skills to do them for you.

  - Examples: status checks, [making changes](./skills/qt), [fixing](./skills/qd), redeploying, etc.
  - Exploit the agent and sub-agent capabilities to build pipelines and workflows. A supervisor agent for orchestrating the work, and sub-agents to do the tasks. GREATLY increases performance for LONG tasks.
- **Building with a specific library:**

  - Use Context7 MCP to let the agent dynamically fetch documentation.
  - For more complex libraries (like LangGraph, which requires architectural decisions and has various patterns), first have the agent do proper research, then ask it to use the `skill-creator` skill to build a comprehensive skill for you. Then use the generated skill to build your application. (Preferably ask the agent to use it.)
- **Periodically clean up your code structure**

  - As you build with AI, the code might get messy - either architecturally, file structure, or code-wise. As your agent to evaluate what is important to you (architecture, tests, etc.) and generate a cleanup plan report. And pass to another agent to implement.
    - Example cleanup workflow: You've built an application with Temporal and have been creating various workflows. First, create a temporal-architect skill using the agent (ask it to do research and build it). Then ask your agent (new session) to lay out all your temporal workflows. Then ask it to read the temporal-architect skill and act as a temporal architect and review your temporal architecture and generate an analysis report. Then (new session) ask it to read the report and the skill, and generate a migration plan to the new improved architecture. Then (new session) ask it to implement the plan. Then (optional but recommended, also a new session) ask to verify the changes, AND create a manual testing guide for you to manually test the impacted features.

Generally, think of the agent as a performant but stupid coder. Not smart enough to make important decisions, but performant enough to do EXACTLY what you say. The key is being "EXACT"!
