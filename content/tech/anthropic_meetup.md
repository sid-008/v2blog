---
Title: Notes from the AI engineers Anthropic Meetup
date: 2026-01-03
description: Notes from the Anthropic AI Engineers Meetup @ PeakXV Blr
---

# Anthropic peakxv meetup

These are notes from the Anthropic AI engineers meetup I attended at the Bangalore PeakXV office(9th Oct 2025).
This is a very rough note based off whatever was being spoken about. If I do get time I will
refine and elaborate on these notes and add in my impressions. Hope this is helpful to whoever reads it!
NOTE: This does not reflect my opinion of LLMs or Anthropic, all opinions are explicitly tagged and mentioned.

Speaker: Daniel Delaney

## Multi agent systems
- Anthropic agent SDK
- single LLM features -> workflows -> agents
- agents vs workflows there's a diff
- agents are LLMs using "tools" dynamically making decisions in their own environment
- open ended problems where you need to iterate
- predicts req num of steps
- multi agents: open ended, parallelisable tasks, lot of info, lot of tools involved
- multi agents: leverage collective intelligence
- Opus as "lead agent" sonnet as "sub agents"
- run tasks for longer -> what does that even mean :skull: more tokens used for them xD

## Case Study: Multiulti agent research system
- agentic search is open ended prob
- multiple pathways to go down
- clear sep of concerns
- parallelise for speed
- need to manage context intelligently
- independent exploration

- Lead agent and as many subagents are required as it thinks
- memory system
- citations agent for eg.
- orchestrator workflow
- similar to single agent system?
- Lead does most of the thinking
- spin up subagents as many as it needs
- subagents iterate as much as required

### Downfall
- coordination complexity
- simple queries resulted in 50 subagents apparently in early systems
- solution: prompt engg leverage

### Solutions:
- Good prompt engineering, think like your agents
- Build sims to understand prompt effects
- early agents were way too verbose
  - selected incorrect tools
- effective prompting requires an accurate mental model of the agent
- scale effort to complexity, don't let agents overuse resources
- the lead agent needs to estimate a good amount to understand where resources go and how

#### Scale effort to query complexity
- Be explicit with what difficulty looks like

#### Teach the orchestrator how to delegate
- explicitly direct subagents about how to operate
- best prompts are frameworks for collaboration, define division of labour and budgets and prob solving approaches

#### Focus on good heuristics rather than rigid rules
- decompose difficult decisions into smaller tasks
- eval source quality(device db -> qdrant)
- Operating heuristicc/ways to learn trumps rigid rules
- Get the agent to _think_ trust the general intelligence of the model
  use good heuristics to instruct
- This has been only possible in the last 6 months, but the trust is hard to keep
  up with.
- tool descriptions matter so that subagents can understand what to use

#### Build agents to improve other agents
- agents are excellent prompt engineers
- trained 1 agents to make tools more effective
- said agent rewrote desc for the tools so subagents could understand it better

#### Evals
- Test multi agent systems early, its apparently very often, small tweaks to the lead agent
  resulted in huge cascading changes
- if they didn't do this it would have been really hard to diagnose issues later on
- don't delay until you can build more through evals

#### LLM as a judge
- LLMs are natural fits for grading outputs when given eval heuristics
- LLM as a judge scales to hundreds of outputs and a range of use cases
- (My personal opinion on this is that their testing methods were incredibly shaky at best)

#### Don't discount human evals
- People testing agents find things that evals miss

#### Costs
- Very expensive
- 15x more tokens
