---
name: agent-onboarding
description: Post-install onboarding flow. Agent greets the user, activates self-compiler, and guides them through configuring CLAUDE.md and workspace files through natural conversation. Use after fresh Claude Code install. Covers identity, goals, style, stack, and generates all config files. The whole flow takes 5-10 minutes.
version: 1.0.0
user-invocable: true
---

# Agent Onboarding

Post-install onboarding skill. Turns a blank Claude Code workspace into a fully
configured agent through a short conversational flow.

## When to activate

- Immediately after the installer finishes
- When the student runs their first `claude` session
- When `/onboarding` is invoked explicitly

## Prerequisites

- Claude Code installed and authenticated
- `~/.claude/` directory exists
- Self-compiler skill available (used internally for knowledge extraction)

## Flow

Seven phases, executed sequentially. Each phase collects information through
natural conversation and generates one or more files.

---

### Phase 1: Greeting (first message)

Send exactly this (localized to detected language):

> Hi! I'm your AI agent, freshly installed and ready to work. But first -- I need
> to know who you are so I can be truly useful.
>
> Let's spend 5 minutes setting me up. I'll ask you questions, and from your
> answers I'll create all the config files I need.
>
> Ready? Let's start with the basics.

No questions in this phase. Wait for any reply to proceed.

---

### Phase 2: Identity (profile.md + USER.md)

**Goal:** Establish who the operator is.

Questions (ask ONE at a time, wait for answer before next):

1. What's your name? How should I address you?
2. What's your timezone? (e.g. UTC+3, PST, "I'm in Berlin")
3. What language do you prefer for communication?
4. What do you do? (role, profession, area of expertise)
5. What's your experience level with AI agents? (beginner / intermediate / advanced)

**Generate from answers:**

- `~/.claude/core/USER.md` -- operator profile (use `templates/user-md-student.md`)
- `~/knowledge/profile.md` -- extended profile with raw answers

**Confirm:** "Got it, {{NAME}}. I'll address you as {{ADDRESS}} and communicate
in {{LANGUAGE}}. Moving on."

---

### Phase 3: Goals & Projects (goals.md)

**Goal:** Understand what the agent will be used for.

Questions:

1. What are you working on right now?
2. What's your main goal for the next month?
3. What kind of tasks will you give me? (code, research, writing, automation...)

**Generate:** `~/knowledge/goals.md`

**Confirm:** "Noted. Main focus: {{ONE_LINE_SUMMARY}}."

---

### Phase 4: Style & Rules (style.md, rules.md)

**Goal:** Learn communication and behavioral preferences.

Questions:

1. How do you like to communicate? (brief answers / detailed explanations,
   formal / casual)
2. Any things I should never do? (e.g. no emoji, no long explanations, never
   delete files without asking)
3. Code style preferences? (language, framework, tabs vs spaces, naming)

**Generate:**

- `~/knowledge/style.md` -- communication style
- `~/knowledge/rules.md` -- hard rules and boundaries

**Confirm:** "Style set: {{BRIEF_STYLE_SUMMARY}}."

---

### Phase 5: Stack & Tools (stack.md)

**Goal:** Map the technical environment.

Questions:

1. What programming languages do you use?
2. What tools and services? (GitHub, Docker, cloud providers, databases)
3. Any APIs or integrations I should know about?

**Generate:** `~/knowledge/stack.md`

**Confirm:** "Stack captured. Primary: {{LANGUAGES}}, {{KEY_TOOLS}}."

---

### Phase 6: CLAUDE.md Generation

**Goal:** Produce the main agent configuration file.

Steps:

1. Load template from `templates/claude-md-student.md`
2. Fill placeholders with collected data:
   - `{{NAME}}`, `{{TIMEZONE}}`, `{{LANGUAGE}}` from Phase 2
   - `{{CODE_STYLE}}` from Phase 4
   - `{{COMMIT_LANGUAGE}}` from Phase 4 (default: English)
   - `{{STYLE_BRIEF_OR_DETAILED}}` from Phase 4 (default: "Brief answers, code first")
3. Present the draft to the student:
   > "Here's your CLAUDE.md draft. Want me to adjust anything?"
4. Apply requested changes
5. Write final version to `~/.claude/CLAUDE.md`

If the student says "looks good" or similar -- write immediately, no further
questions.

---

### Phase 7: Completion

Send a completion message listing the actual files created with their full paths:

> Setup complete! Here's what I created:
> - `~/.claude/CLAUDE.md` -- my operating rules
> - `~/.claude/core/USER.md` -- your profile
> - `~/knowledge/profile.md` -- extended identity
> - `~/knowledge/goals.md` -- your goals
> - `~/knowledge/style.md` -- communication preferences
> - `~/knowledge/rules.md` -- boundaries
> - `~/knowledge/stack.md` -- tech stack
>
> I'm ready to work. What's our first task?

Only list files that were actually created (skip phases that were skipped).

---

## Rules

- **One question at a time.** Never send a wall of questions. Ask, wait, process,
  ask next.
- **Confirm after each answer.** One line showing what was captured. The student
  must see that the agent understood correctly.
- **"Skip" means defaults.** If the student says "skip", "idk", or gives a
  minimal answer -- use sensible defaults and move on. Do not push.
- **Minimal answers are fine.** If the student gives one-word answers -- work with
  what you have. The onboarding should never feel like an interrogation.
- **Files in English.** All generated config files use English for token
  optimization, regardless of the student's language preference.
- **Communicate in student's language.** After Phase 2, switch to the language
  the student chose.
- **5-10 minutes max.** If the flow drags, compress remaining phases: combine
  questions, use defaults, finish fast.
- **Self-compiler internally.** Use the self-compiler skill to extract structured
  knowledge from free-form answers. Do not mention self-compiler to the student.
- **Idempotent.** If onboarding is run again, overwrite existing files after
  confirming with the student.
- **No sensitive data.** Never ask for passwords, API keys, or tokens during
  onboarding. Those go into `.env` files separately.

## Defaults (when student skips)

| Field | Default |
|---|---|
| Address | First name |
| Timezone | UTC |
| Language | English |
| Experience | Beginner |
| Code style | Standard conventions for detected language |
| Commit language | English |
| Autonomy | Green: code, tests, git branch. Red: delete, deploy, force push |

## File structure created

```
~/.claude/
  CLAUDE.md              # Main agent config
  core/
    USER.md              # Operator profile
~/knowledge/
  profile.md             # Extended identity
  goals.md               # Current projects and goals
  style.md               # Communication preferences
  rules.md               # Hard rules and boundaries
  stack.md               # Tech stack and tools
```

## Integration with self-compiler

The self-compiler skill is activated during Phase 2-5 to:

1. Parse free-form answers into structured key-value pairs
2. Detect implicit preferences (e.g. student writes casually -> set style=casual)
3. Resolve conflicts (e.g. says "brief" but gives long answers -> ask to clarify)
4. Generate consistent markdown from heterogeneous input

The agent calls self-compiler internally. The student sees only natural
conversation, never the compilation process.
