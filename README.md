# Multi-Agent Coordination
**Session Time:** 120 minutes  
**Format:** Instructor-Led  

---

## Prerequisites

- Completion of Sessions 1–8  
- Understanding of the single-agent loop: reasoning, memory, tools, and feedback (Sessions 2–8)  
- Familiarity with tool-call orchestration and human-in-the-loop patterns (Session 7)  
- Understanding of how the system prompt defines an agent's role, goals, and constraints (Session 6)  

---

## Learning Objectives

By the end of this session, students will be able to:

1. **Describe** different models of multi-agent coordination, such as sequential, cooperative, and hierarchical.
2. **Simulate** a scenario where multiple agents work together on a shared task, highlighting the interaction flow between agents.
3. **Analyze** the risks of miscommunication and conflict between agents and **propose** coordination or conflict-resolution mechanisms to reduce failures.

> A single agent can do a great deal. But many of the most valuable workflows — research, analysis, content production — are too complex or too multi-disciplinary for a single agent to handle reliably. Multi-agent systems are the architectural response to that limitation.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Why Multiple Agents? Coordination Models | 20 min |
| 2 | Simulating a Multi-Agent Workflow | 25 min |
| 3 | Risks, Failures, and How to Handle Them | 20 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Why Multiple Agents? Coordination Models (20 min)

### Why Not Just One Agent?

Before diving into how multi-agent systems work, it's worth asking: why would you ever need more than one agent?

Think of it like a team at work. A single person can handle a simple task alone. But a complex project — say, launching a product — requires a researcher, a designer, a writer, and a project manager. Assigning all of that to one person produces mediocre results across the board. Specialization works.

The same logic applies to agents. Three limitations of single-agent systems motivate multi-agent design:

<img width="924" height="275" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/a4f5baab-ed68-4c2d-ad5e-78ad7cef5e02" />


> **Important:** Multi-agent is not always the right answer. It adds coordination complexity and more ways things can go wrong. The decision to use multiple agents should be justified by one of the three reasons above — not just because it sounds more powerful.

---

### Coordination Model 1: Sequential (The Pipeline)

Agents form a chain. Each agent completes its task and passes the result to the next. No agent starts until the previous one has finished.

```
[Agent A: Research]
       ↓ output
[Agent B: Analysis]
       ↓ output
[Agent C: Report Writing]
       ↓
[Final Deliverable]
```

**Think of it like:** a relay race. Each runner waits for the baton before starting. Clear, predictable, easy to follow.

**When it works well:** When each step truly depends on the full output of the step before it — content pipelines, document drafting, staged analysis.

**The main risk:** If Agent A makes a mistake, Agents B and C build on top of that mistake. The error travels through the whole chain.

---

### Coordination Model 2: Hierarchical (The Manager + Team)

An orchestrator agent receives the top-level goal, breaks it into subtasks, delegates to worker agents, collects their results, and combines them into a final output.

```
         [Orchestrator Agent]
        /         |          \
[Worker A]  [Worker B]  [Worker C]
 Research    Analysis    Writing
     ↓            ↓           ↓
         [Orchestrator]
         Combines results
               ↓
       [Final Deliverable]
```

**Think of it like:** a project manager directing a team. The manager doesn't do the research or write the report — they coordinate, check in, and pull everything together.

**When it works well:** Complex tasks that can be broken into parallel workstreams handled by different specialists.

**The main risk:** If the orchestrator reasons incorrectly about how to split the task, all the downstream work goes in the wrong direction.

---

### Coordination Model 3: Cooperative (The Peer Team)

Agents work as equals — no central orchestrator. They communicate with each other directly, sharing information and results as the workflow evolves.

```
[Agent A: Data Collection] ←→ [Agent B: Validation]
          ↕                              ↕
[Agent C: Analysis]        ←→ [Agent D: Reporting]
```

**Think of it like:** a cross-functional team without a designated project manager, where everyone coordinates directly with whoever they need.

**When it works well:** Tasks where agents are genuinely interdependent and need to check each other's work in real time. More resilient — one agent failing doesn't necessarily stop all the others.

**The main risk:** Much harder to design, trace, and debug. When agents communicate freely, it's difficult to understand what went wrong and why.

---

### Coordination Model Summary

<img width="916" height="208" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/e2307738-25ae-4a7d-9f31-6797e044fb80" />


### Discussion Prompt (2 min)

> Think about a real workflow at your job — something that involves multiple people or steps. Which coordination model does it most resemble? Sequential? Hierarchical? Cooperative?

---

## 2. Simulating a Multi-Agent Workflow (25 min)

### The Scenario: Weekly Competitive Intelligence Report

A business development team needs a weekly summary of what their three main competitors have been doing — recent news, product changes, and what it all means strategically.

This task involves three very different types of work: **research**, **analysis**, and **writing**. That's a natural fit for a hierarchical multi-agent system.

**The agent team:**

<img width="920" height="293" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/4d4e942e-3014-4ad9-9dbc-c98c0ca54405" />


---

### Step-by-Step: How the Workflow Runs

#### Step 1: Orchestrator receives the goal and delegates

```
Goal arrives:
"Generate this week's competitive intelligence report for Competitors X, Y, and Z."

Orchestrator thinks:
"This needs three types of work: research, analysis, and writing.
 Research must happen first. Analysis depends on research.
 Writing depends on analysis.
 I'll run research for all three competitors at the same time,
 then pass the results to the Analysis Agent."

Orchestrator delegates:
→ Research Agent: "Find all news and product changes for Competitor X in the past 7 days."
→ Research Agent: "Do the same for Competitor Y."
→ Research Agent: "Do the same for Competitor Z."
[All three run at the same time]
```

#### Step 2: Research Agent gathers information (three parallel instances)

```
Research Agent — Competitor X:

Searches for: "Competitor X news last 7 days"
Searches for: "Competitor X product pricing changes"
Checks their press releases

Returns to Orchestrator:
- News summary: "Competitor X raised $85M and announced expansion into enterprise."
- Product change: "New pricing tier launched — 15% cheaper for mid-market customers."
- Sources: TechCrunch, Competitor X blog
```

*The same process runs simultaneously for Competitors Y and Z.*

#### Step 3: Orchestrator routes research to the Analysis Agent

```
Orchestrator:
"All three research summaries are in.
 Sending them to the Analysis Agent with the internal pricing data."

Analysis Agent receives:
- Research summaries for X, Y, and Z
- Internal data about the company's own pricing
- Last week's report for comparison
```

#### Step 4: Analysis Agent interprets and draws conclusions

```
Analysis Agent thinks:
"Both X and Y reduced mid-market pricing this week.
 Let me compare that to our own mid-market pricing."

Checks internal data → finds: our price is now 18% above Competitor X.

Conclusion:
"This is significant. We are now noticeably more expensive than our two
 closest competitors in the mid-market segment. This creates churn risk
 before Q2. A pricing review should be recommended."

Returns to Orchestrator:
- Key finding: pricing gap of 15-18% vs. X and Y
- Strategic implication: churn risk in mid-market
- Recommendation: pricing review before Q2
```

#### Step 5: Writing Agent produces the final report

```
Writing Agent receives:
- Research summaries
- Analysis output
- Standard report template

Writes:
- Executive summary (leads with the pricing story — most important finding)
- Competitor-by-competitor section (facts + implications)
- Recommendations section

Returns: Completed report document
```

#### Step 6: Orchestrator delivers

```
Orchestrator:
"Report is complete. Sending to the business development team."

→ Report delivered via email
→ Summary message: "Key finding this week: mid-market pricing gap of 15–18%
                    vs. Competitors X and Y. Pricing review recommended before Q2."
```

---

### What This Simulation Reveals

Walking through this workflow step by step makes visible the design decisions that determine whether the system works:

<img width="934" height="311" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/7bef3105-5385-4f20-8f76-d52a64c7060b" />


---

## 3. Risks, Failures, and How to Handle Them (20 min)

### Multi-Agent Systems Fail Differently

Single-agent failures are relatively straightforward: the agent reasons incorrectly, calls the wrong tool, or produces a bad output. Multi-agent failures are more complex because they can emerge from the *interaction between agents* — not just from any individual agent's behavior.

---

### Failure 1: Cascading Errors

In a sequential or hierarchical system, an error in an upstream agent travels to every agent below it. Each subsequent agent treats the incorrect output as fact and builds on it.

```
Research Agent returns wrong pricing data
         ↓
Analysis Agent calculates implications from wrong numbers
         ↓
Writing Agent produces recommendations based on flawed analysis
         ↓
Team receives a report with incorrect conclusions —
and has no visibility into where the error started
```

**Why it's dangerous:** By the time the error reaches the end of the chain, it has been transformed and amplified. The original mistake is invisible in the final output.

**How to handle it:**  
Add a **checkpoint** between agents for high-risk transitions. Before passing output downstream, a simple check verifies that the output is complete and in the expected format. If it fails the check, the output goes back to the previous agent with an explanation of what's missing.

```
Research Agent output →
[Checkpoint: Are all required fields present? Is the data in the expected format?]
→ PASS: send to Analysis Agent
→ FAIL: return to Research Agent with description of what's missing
```

---

### Failure 2: Lost Context at Handoff

When one agent passes its output to the next, critical background context can get lost in translation. The receiving agent gets the data but not the context needed to interpret it correctly.

```
Research Agent sends:
"Competitor X announced enterprise expansion."

Analysis Agent receives this without background context and reasons:
"Enterprise expansion is a standard growth move. Low strategic urgency."

What the Analysis Agent didn't know:
Our company gets 78% of its revenue from enterprise customers.
Competitor X entering enterprise is a direct existential threat — not a routine move.
```

**Why it happens:** Each agent is focused on its own task. It doesn't know what the next agent needs to know in order to do its job well.

**How to handle it:**  
Define a **standard handoff format** that includes not just the output, but the context needed to interpret it — the original goal, relevant constraints, and any flags the next agent should be aware of.

```
Handoff from Research Agent to Analysis Agent:
- Task completed: Research on Competitor X
- Finding: Enterprise expansion announced
- Context for Analysis Agent: Our company is 78% enterprise-dependent.
  This development has high strategic relevance — flag as priority.
- Original goal reminder: Identify competitive threats to our enterprise business
```

---

### Failure 3: Conflicting Outputs

In cooperative systems — or in hierarchical systems where multiple workers run in parallel — agents can produce outputs that contradict each other. Without a way to resolve the conflict, the system gets stuck or produces a confused final result.

```
Analysis Agent A concludes:
"Competitor X's funding signals aggressive expansion —
 immediate pricing response recommended."

Analysis Agent B concludes:
"Competitor X's funding is standard for this stage —
 monitor but no action needed yet."

Orchestrator receives both. No resolution defined.
Result: The orchestrator either picks one arbitrarily or produces no recommendation.
```

**How to handle it:**  
Define in advance how conflicts get resolved. The simplest options:

| Resolution approach | How it works | Best for |
|---|---|---|
| **Orchestrator decides** | The orchestrator agent receives both outputs and reasons to a final decision | Complex or subjective decisions |
| **Human escalation** | Conflict is flagged to a human reviewer | High-stakes decisions where machine resolution is insufficient |
| **Most conservative wins** | When in doubt, choose the option that carries less risk | Risk-sensitive contexts |

The key principle: **don't leave conflict resolution undefined**. An unresolved conflict always produces a worse outcome than a defined resolution protocol — even an imperfect one.

---

### Failure 4: The Orchestrator Gets It Wrong

In hierarchical systems, the orchestrator is the single point of failure for task decomposition. If it misjudges how to split the task, all the downstream work goes in the wrong direction — even if every worker agent executes perfectly.

```
Orchestrator misreads the goal:
"Generate a competitive intelligence report"
→ Orchestrator interprets this as a market size report
→ Delegates research on market growth rates, not competitor activities

Workers do exactly what they were asked.
Final output: a market analysis, not a competitive intelligence report.
```

**How to handle it:**  
Two practices reduce orchestrator errors significantly:

1. **Be explicit in the goal**: The instruction given to the orchestrator should spell out what success looks like, not just what the task is. *"Generate a competitive intelligence report covering: recent news, product changes, pricing movements, and strategic recommendations."*

2. **Have the orchestrator confirm its plan before acting**: Prompt the orchestrator to describe how it plans to decompose the task before delegating. A human can review and correct the plan before any workers are called.

---

### The Coordination Design Checklist

Before deploying a multi-agent system, verify:

<img width="703" height="399" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/8fd4bd79-4037-4094-8fb5-bc766480c7f8" />

---

## 4. Break (10 min)

---

## 5. Lab (60 min)

*Lab details in the Lab Guide document.*

---

## 6. Wrap-Up & Reflection (5 min)

### Key Takeaways
- Specialize Agents Like You'd Specialize a Team
- The Handoff Is as Important as the Work
- Design for Failure, Not Just for Success

---

## Resources

- [CrewAI — Multi-Agent Coordination Documentation](https://docs.crewai.com/concepts/crews)
- [LangGraph — Multi-Agent Workflows](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi_agent_collaboration/)
- [AutoGen — Microsoft Multi-Agent Framework](https://microsoft.github.io/autogen/)
- [Flowise — Multi-Agent Flow Setup](https://docs.flowiseai.com/)
- Lilian Weng — *LLM-powered Autonomous Agents: Planning* (2023) — [lilianweng.github.io](https://lilianweng.github.io/posts/2023-06-23-agent/#planning)
- OWASP — *Top 10 for LLMs: Excessive Agency* (2023)

---

*General Assembly · Build AI Agents · BAA 16.0 · © 2026*
