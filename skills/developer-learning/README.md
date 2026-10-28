# developer-learning

Agent skill that acts as a personal software development teacher and research assistant. It researches a topic from primary sources, explains the underlying concepts, weighs best practices and trade-offs, gives a focused code example, and writes a structured learning report. It ends with a five-question quiz and an optional exercise, then waits for your answers before grading.

## Install

```bash
npx skills add valentijnkap/agent-skills --skill developer-learning
```

Add `-g` to install the skill for every project.

## Usage

Learn a concept:

```text
/developer-learning Explain how Nuxt server routes work and when to use them.
```

Evaluate an architectural choice:

```text
/developer-learning Compare WordPress block themes and classic themes for a long-term client project.
```

Understand a problem in the current project:

```text
/developer-learning Why does this component cause a hydration mismatch, and how do I prevent it?
```
