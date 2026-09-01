# GitHub Learning Repository Setup

Create a clean, professional GitHub learning repository structure for a 4th-year Computer Science student preparing for software engineering roles.

## Goal

I want to use this repository as my **personal learning and revision system**. It should contain my own notes, explanations, implementations, examples, mistakes, and practice—not copied tutorials.

Keep the structure simple, scalable, and easy to maintain.

## Repository Structure

Create the following folders:

```text
learning-journey/
│
├── 01-DSA/
│   ├── Arrays/
│   ├── Strings/
│   ├── Linked-List/
│   ├── Stack-Queue/
│   ├── Recursion-Backtracking/
│   ├── Binary-Search/
│   ├── Trees/
│   ├── Graphs/
│   └── Dynamic-Programming/
│
├── 02-CS-Fundamentals/
│   ├── OOP/
│   ├── DBMS/
│   ├── Operating-Systems/
│   └── Computer-Networks/
│
├── 03-Development/
│   ├── HTML-CSS/
│   ├── JavaScript/
│   ├── TypeScript/
│   ├── React/
│   ├── NodeJS/
│   ├── Express/
│   ├── Databases/
│   └── APIs/
│
├── 04-System-Design/
│   ├── Fundamentals/
│   ├── HTTP-REST/
│   ├── Caching/
│   ├── Load-Balancing/
│   ├── Databases/
│   ├── Scalability/
│   └── Case-Studies/
│
├── 05-Tools-DevOps/
│   ├── Git-GitHub/
│   ├── Linux/
│   ├── Docker/
│   └── Cloud/
│
├── 06-Misc-Learnings/
│   ├── AI/
│   ├── New-Technologies/
│   └── Useful-Concepts/
│
└── README.md
```

## README Requirements

Create a professional root `README.md` containing:

1. Short introduction
2. Purpose of the repository
3. Learning roadmap
4. Main subjects
5. Current learning status
6. Links to each major section
7. Progress checklist
8. Important rule: notes and implementations are written from my own understanding
9. A simple "Last Updated" section

Do NOT make the README unnecessarily flashy. Keep it professional and suitable for a software engineering portfolio.

## Structure Inside Topics

For important concepts, use Markdown files such as:

```text
Topic/
├── README.md
├── concepts.md
├── examples/
└── practice/
```

However, **do not create unnecessary files or folders for every topic**. Keep the repository lightweight.

Each concept note should generally follow this structure:

```markdown
# Topic Name

## What is it?

## Why is it important?

## Core Concepts

## Example

## Implementation

## Time & Space Complexity
<!-- where applicable -->

## Common Mistakes

## Interview Questions

## What I Learned

## Practice
```

Only include sections that make sense for the topic.

## Important Rules

- Do not add fake learning content.
- Do not generate hundreds of empty files.
- Do not copy content from tutorials.
- Do not over-engineer the repository.
- Keep naming consistent.
- Use Markdown for conceptual notes.
- Use appropriate languages for implementations.
- Keep code readable and beginner-friendly.
- Add `.gitignore` where appropriate.
- Add a useful root `README.md`.
- Make the structure easy to extend later.
- Prioritize maintainability over complexity.

## GitHub Philosophy

This repository should represent my **actual learning journey**, not a collection of copied notes.

The workflow should be:

Learn → Understand → Practice → Document → Commit → Apply in Projects.

Before creating anything, inspect the existing workspace. If files already exist, **do not delete or overwrite them**. Integrate the new structure safely.

After setup, show me:

1. The final directory tree
2. Files created
3. Any existing files left untouched
4. Recommended first 5 things I should add
5. Recommended Git commit message for the initial setup