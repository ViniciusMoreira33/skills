---
name: initial-prompt-builder
description: Turns a rough first request into a complete, well-structured prompt before any work starts, so the task is done right the first time without extra follow-up prompts. Use this skill at the start of any new task or initial prompt that has a real deliverable (writing, analysis, code, research, design, documents, plans, data work), including when the user says "new task", "I need", "help me with", "create", "build", "write", "analyze", or pastes a rough idea, brief, or half-formed request. Also use it when the user asks to improve, complete, or prepare a prompt for Claude or another AI. Skip it for quick factual questions, small talk, and follow-up messages inside a task that is already in progress.
---

# Initial prompt builder

The first prompt of a task decides how many prompts the task takes. A request that leaves out the goal, the format, or the constraints gets a generic answer, and the user pays for it with correction rounds. This skill closes those gaps up front. It reads the rough request, fills in what the context already answers, asks only what would change the result, and then works from a complete brief.

The method comes from Anthropic's "Prompting best practices" page for current Claude models (checked November 2026). The same brief also works as a prompt for other AI tools.

## When to use it and when to skip it

Use it when the user starts a task with a real deliverable and the request is short, vague, or missing context. Skip it and just answer when:
- the question is a quick fact, a definition, or a one-step lookup
- the message continues a task already in progress
- the request already states the goal, the deliverable, the constraints, and the context
- the user says "just do it", "no questions", or similar. That instruction wins, so state assumptions briefly and proceed.

Keep the overhead proportional. A small task gets a two-line brief and a large task gets a full one. The skill should never add a turn the task did not need.

## Workflow

### 1. Read what is already known

Use the message, attached files, the conversation so far, any project or memory context, and available tools. Anything those answer is not a question for the user.

### 2. Run the gap check

Check the request against these eight elements. Mark each as covered, inferable, missing but minor, or missing and material. Material means a wrong guess would send the work in a different direction.

1. Goal and reason. What the user is trying to achieve and why. The reason lets the model make good choices the prompt did not anticipate.
2. Deliverable. What gets produced, in which format, at what length, in which language.
3. Audience and use. Who reads or uses the result and where it goes next.
4. Inputs. Documents, data, links, code, prior work, and limits of the environment.
5. Constraints. Tone, tools, scope limits, things to avoid, deadlines, and the reason behind each.
6. Scope. Whether the user wants exactly what they asked for or more, such as extra features, depth, or alternatives.
7. Success check. How the user will judge the result and what a finished result looks like.
8. Examples. A sample of the desired output or a reference to match, when format, tone, or structure matter.

### 3. Decide whether to proceed or ask

- No material gaps: go to step 4 and state the assumptions you made.
- Material gaps: ask one batched round with at most three questions. Put a recommended default in each question so the user can reply "ok" or pick an option. If a tappable-options tool is available, use it. Otherwise use short numbered questions.
- Never ask a second round. After the answers, or if the user skips them, proceed with the best defaults and label them as assumptions.

### 4. Build the brief

Write the complete prompt using the construction rules below. Include only the parts the task needs.

### 5. Deliver

Default mode runs the task. Show a short block first, then do the work:

```
Understood as: one or two sentences on the goal and the deliverable.
Assumptions: the defaults you chose, one line each, so the user can correct them in one reply.
```

Do not show the full internal prompt unless the user asks for it.

Prompt mode applies when the user wants a prompt to use elsewhere, such as another AI, a Claude Project, Claude Code, or an API call. Output the finished prompt in a code block ready to paste, followed by one line listing the assumptions. Do not run the task.

Write the brief in the language the user wrote in, unless the destination needs another language.

## Construction rules

Apply the rules that fit the task.

**Be direct and complete.** The test is whether a skilled new hire with no context could follow the prompt without asking what you meant. If they would be confused, the model will be too.

**Ask for more than the minimum when the user wants it.** A bare "create an analytics dashboard" gets a basic one. Name the filters, charts, and interactions needed, or say that a fully featured result beyond the basics is wanted.

**Give the reason behind each constraint.** "Avoid ellipses because a text-to-speech engine will read this aloud" lets the model apply the logic to cases the prompt did not list. State what to do instead of what to avoid. "Write in flowing prose paragraphs" works better than "do not use markdown".

**Add examples when format, tone, or structure matter.** Use three to five, keep them close to the real use case, and vary them so the model does not copy an accidental pattern. Wrap each in an <example> tag inside <examples>.

**Separate sections with XML tags** when the prompt mixes instructions, context, examples, and input. Use descriptive names and keep them consistent, such as <instructions>, <context>, and <input>. Nest tags when the content has a hierarchy.

**Place long material first.** For inputs over about 20,000 tokens, put the documents at the top and the task or question at the end. In Anthropic's tests, ending with the query improved quality by up to 30 percent, mostly on multi-document inputs. Wrap each document in <document> tags with <source> and <document_content> subtags. For long documents, ask the model to copy the relevant quotes into tags before it analyzes them.

**Set a role in one sentence** when it helps focus tone and behavior, for example "You are a senior financial analyst reviewing a small-business budget."

**Match the verb to the intent.** "Can you suggest changes" gets suggestions, and "change this function to improve performance" gets edits.

**Guide reasoning with goals, not scripts.** "Think thoroughly" plus a clear success criterion usually beats a hand-written list of deduction steps. Use numbered steps only when the order of actions matters. For code, math, and numbers, add a closing line such as "before you finish, verify the result against these criteria". Do not ask the model to write out its reasoning in thinking tags, and do not set manual thinking budgets.

**Use plain emphasis.** Skip all caps and words like "CRITICAL" and "MUST". Current models follow instructions as written, and heavy emphasis can make them overuse tools or overdo a rule. Normal wording such as "Use this tool when..." works.

**Describe the output you want, and match the prompt's own style to it.** A prompt written without markdown tends to produce less markdown.

## Add-ins for specific task types

Include these only when the task matches.

**Front-end and design work.** Without direction, models produce generic layouts. Specify typography (distinctive fonts, not defaults like Arial or Inter), a committed color palette defined with CSS variables, motion (for example one staged page-load animation), and backgrounds with depth instead of a flat color. List what to avoid, such as purple gradients on white. Name the user's own fonts and colors when they have them.

**Code and agentic work.** State the scope ("make only the requested change, no extra abstractions or refactors"). Ask the model to read the relevant files before answering questions about code, to write a general solution instead of one that hard-codes the test cases, to remove any temporary files it creates, and to confirm before destructive or hard-to-reverse actions such as deleting files or force-pushing.

**Research.** Define what a successful answer contains, ask for verification across several sources, and ask the model to track its confidence in its notes.

**Long runs.** Say how to save progress and state so the work can resume in a fresh context window.

## When the prompt is for the API

Anthropic keeps a separate prompting page for each model, so read the one for the target model. Three version limits matter for the prompt and its request settings:
- Prefilled assistant turns on the last message are rejected from Claude 4.6 on. Use a direct instruction or structured outputs.
- `budget_tokens` returns an error from Claude 4.7 on. Use adaptive thinking with an effort setting.
- On Claude 5.1 and later, keep the message history append-only and pass thinking blocks back unchanged. Put changes in a mid-conversation system message or in server-side context management, never by editing earlier turns.

These limits change with new models, so check the current page before relying on them.

## Template

Drop any section the task does not need.

```
<role>One sentence, if useful.</role>

<context>
Why the task exists, who the audience is, and background the model lacks.
</context>

<input>
Documents or data, placed first when long.
</input>

<instructions>
The task, the deliverable, and what a finished result looks like.
Constraints, each with its reason, stated as what to do.
Output format.
</instructions>

<examples>
<example>...</example>
</examples>

The specific request, last.
```

## Example

Rough request: "make me a presentation about our Q3 results"

Gap check: the data source is missing and material, because no file is attached. The audience is missing and material, because executives and a team need different depth. The length is missing but minor.

One batched round:
1. Can you attach the Q3 data or summary?
2. Who is the audience? (default: the leadership team)
3. How long? (default: 8 to 10 slides)

After the answers, deliver:

```
Understood as: a 10-slide Q3 results deck for the leadership team, built from the attached data.
Assumptions: lead with revenue and margin, one chart per slide, finish with next-quarter risks.
```

Then build the deck from a brief that includes the data in <input>, the audience and purpose in <context>, and the slide structure and chart rules in <instructions>.

## Principles

- Ask only what the context cannot answer, and ask once.
- Prefer a stated assumption over a question when a wrong guess is cheap to fix.
- Keep the brief as short as the task allows. Do not pad it.
- Do not explain prompting theory to the user unless they ask.
- Do not rewrite a prompt that is already complete.
