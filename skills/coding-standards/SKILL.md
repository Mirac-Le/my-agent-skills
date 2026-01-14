---
name: coding-standards
description: Personal coding standards and interaction preferences. Apply when writing code, debugging, reviewing, or making git commits. Ensures consistent code quality, communication style, and prevents over-engineering.
---

# Coding Standards

## Response Style

- Reply in Chinese for explanations, English for code
- Be concise; avoid unnecessary repetition or filler
- Don't apologize for errors—fix them directly
- Add TODO comments for incomplete sections
- When uncertain, ask clarifying questions

## Code Generation

- Always include necessary imports
- Use type hints for all function signatures
- Prefer explicit over implicit behavior
- Show minimal diffs when modifying existing code
- Include brief inline comments for complex logic

## Debugging Approach

- Ask for error messages and stack traces first
- List top 3 likely causes before investigating
- Provide multiple potential solutions when applicable

## Code Review Priority

1. Security vulnerabilities
2. Correctness issues
3. Performance problems
4. Style improvements

## Git Commits

- Use Conventional Commits format
- Keep subject under 72 characters
- Include scope when relevant (e.g., `feat(api): add endpoint`)

## Constraints

- Only make changes directly requested
- No "improvements" beyond the ask
- No extra error handling for impossible scenarios
- No backwards-compatibility shims when you can just change code
- A bug fix doesn't need surrounding code cleaned up

## Confirmation Required

- Ask before executing destructive operations (delete, truncate, drop)
- Confirm when encountering ambiguous requirements
- Do NOT skip steps silently—report issues and ask how to proceed
- When discovering potential bugs or inconsistencies, report and ask before fixing
