# Agents Manifest

Paths and descriptions for agent-related files and directories in the project.

# 🗺️ PROJECT MAP 
Root: `/` | Config: `.agents/` | Context: `project/`

## 📂 Core Directories
- **Agents Logic:** `.agents/` (Definitions, Prompts, Skills)
- **Project Context:** `project/` (PRD, ERM, ADD, Roadmap)
- **Execution Tracking:** `project/specs/` (Specs & Plans)

## 📍 Key References
- **Agents Rules:** `AGENTS.md`
- **Requirements:** `project/PRD.md`
- **Architecture:** `project/ADD.md` & `ERM.md`
- **Current Task:** `project/specs/*.plan.md`
- **Progress:** `project/ROADMAP.md` & `CHANGELOG.md`

## ⚡ Workflow Rules
1. Check `ROADMAP.md` before starting.
2. Follow `*.spec.md` & update `*.plan.md` in real-time.
3. Log major changes in `CHANGELOG.md`.

```txt
/                            # Root directory of the project
├── AGENTS.md                # General documentation about agents in the project
├── CHANGELOG.md             # Main project changelog (released features and changes)
├── README.md                # Main project README
├── .agents/                 # Agent configuration folder
│   ├── manifest.md          # Path dictionary (this file)
│   ├── *.agent.md           # Specific agent definitions
│   ├── prompts/             # Reusable prompts directory
│   └── skills/              # Skill definitions directory
└── project/                 # Project documentation (Product and architecture)
    ├── ADD.md               # Architecture Design Document
    ├── briefing.md          # Project briefing
    ├── ERM.md               # Entity-Relationship Model
    ├── PRD.md               # Product Requirements Document
    ├── ROADMAP.md           # Product Roadmap and Milestones
    └── specs/               # Specifications and plans
        ├── *.spec.md        # Detailed specifications for features 
        └── *.plan.md        # Implementation plans for features
```