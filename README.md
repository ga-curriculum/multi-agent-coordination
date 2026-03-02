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

> A single agent can do a great deal. But many of the most valuable workflows — research, analysis, content production, software development — are too complex, too long, or too multi-disciplinary for a single agent to handle reliably. Multi-agent systems are the architectural response to that limitation.

---

## Session Breakdown

| # | Segment | Duration |
|---|---------|----------|
| 1 | Models of Multi-Agent Coordination | 15 min |
| 2 | Simulating a Multi-Agent Workflow | 20 min |
| 3 | Risks, Failures, and Coordination Mechanisms | 10 min |
| 4 | Break | 10 min |
| 5 | Lab | 60 min |
| 6 | Wrap-Up & Reflection | 5 min |
|   | **Total** | **120 min** |

---

## 1. Models of Multi-Agent Coordination (15 min)

### Why Multiple Agents?

Before introducing coordination models, it is worth establishing why multi-agent systems exist at all — because the answer shapes every architectural decision that follows.

Three limitations of single-agent systems motivate multi-agent design:

<img width="917" height="293" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/46413759-9ae8-46dd-9777-995db63cfcaf" />


Multi-agent architecture is not always the right answer. It introduces coordination overhead, failure surfaces, and debugging complexity that a single well-designed agent avoids. The decision to use multiple agents should be justified by one or more of the three motivations above.

### Coordination Model 1: Sequential

In sequential coordination, agents form a pipeline. Each agent completes its task and passes its output to the next agent as input. No agent begins until the previous one has finished.

<img width="934" height="121" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/12366b5b-61b5-40bf-b75e-5a897ecd87e5" />


**Characteristics:**
- Simple to implement and debug — execution order is deterministic
- Each agent's input is well-defined (the output of the previous agent)
- Total execution time is the sum of all agents' execution times — no parallelism
- A failure at any stage blocks all downstream agents

**Best suited for:** Workflows where each step depends on the full output of the previous step — content pipelines, document processing, staged analysis.

---

### Coordination Model 2: Hierarchical

In hierarchical coordination, an orchestrator agent directs one or more worker agents. The orchestrator receives the top-level goal, decomposes it into subtasks, delegates each subtask to the appropriate worker, collects results, and synthesizes the final output.

<img width="935" height="331" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/f00ed5a6-936f-43cf-9a15-f735d45a1985" />


**Characteristics:**
- The orchestrator maintains the full task context; workers maintain only their subtask context
- Workers can run in parallel if their subtasks are independent
- The orchestrator is a single point of failure — if it reasons incorrectly about task decomposition, all downstream work is misdirected
- Debugging requires understanding both the orchestrator's decomposition logic and each worker's execution

**Best suited for:** Complex tasks with multiple independent subtasks, workflows requiring coordination across specialized domains, tasks where the decomposition logic needs to adapt based on intermediate results.

---

### Coordination Model 3: Cooperative (Peer-to-Peer)

In cooperative coordination, agents work as peers without a central orchestrator. They communicate directly with each other, sharing information and results as the workflow progresses. Coordination emerges from defined communication protocols rather than a hierarchy.

<img width="412" height="191" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/9d606f8d-5a4e-4886-bad7-8cec20b9b3d9" />


**Characteristics:**
- No single point of failure in the coordination layer — the failure of one agent does not block all others
- More resilient to individual agent failures than hierarchical coordination
- Significantly more complex to design and debug — emergent communication patterns are harder to trace
- Risk of conflicting outputs if agents make inconsistent assumptions about shared state

**Best suited for:** Systems that require high resilience, workflows where tasks are genuinely interdependent and benefit from peer feedback, simulations and research environments.

---

### Coordination Model 4: Consensus-Based

A specialized form of cooperative coordination where multiple agents independently evaluate the same problem and a consensus mechanism determines the final output. Used when reliability and accuracy are more important than speed.

<img width="401" height="212" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/2a5f104d-0ac7-4318-9083-90089a80d42b" />


**Characteristics:**
- High reliability — errors by any single agent are less likely to propagate
- Computationally expensive — multiple agents process the same input
- Adds latency — consensus takes time
- Appropriate for high-stakes decisions where the cost of a wrong answer exceeds the cost of extra computation

**Best suited for:** Fact-checking, safety-critical decision support, outputs that will be presented to external stakeholders without human review.

### Coordination Model Summary

<img width="932" height="284" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/e68a9f1a-839a-4b3f-a24f-c983e1fb086c" />


---

## 2. Simulating a Multi-Agent Workflow (20 min)

### The Scenario: Competitive Intelligence Report

A business development team needs a weekly competitive intelligence report on three key competitors. The report must include: recent news and announcements, product and pricing changes, analysis of strategic implications, and an executive summary with recommended actions.

This task is too long and multi-disciplinary for a single agent to handle reliably. It is an ideal candidate for a hierarchical multi-agent system.

**The agent team:**

<img width="925" height="374" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/353fce07-5382-4943-92ed-50d6739d894e" />


### Step-by-Step Interaction Flow

#### Step 1: Orchestrator Receives Goal and Decomposes Task

```
[Input to Orchestrator]
Goal: "Generate this week's competitive intelligence report for Competitors X, Y, Z."
Deadline: End of business today.
Format: Standard CI report template.

[Orchestrator Reasoning]
THINK: "This requires three types of work: research, analysis, and writing.
        Research must complete before analysis can begin.
        Writing can begin once analysis is complete.
        I will run research for all three competitors in parallel,
        then pass all results to the Analysis Agent,
        then pass analysis to the Writing Agent."

[Orchestrator Output — Delegations]
→ Research Agent: "Find all news, announcements, and product changes for 
                   Competitor X in the past 7 days. Return structured summary."
→ Research Agent: "Same for Competitor Y."
→ Research Agent: "Same for Competitor Z."
[Three parallel calls — Research Agent handles each independently]
```

#### Step 2: Research Agent Executes (Three Parallel Instances)

```
[Research Agent — Competitor X]

OBSERVE: "Task: find recent news and product changes for Competitor X."
THINK:   "I'll search for news first, then check their product page for changes."

ACT: → web_search("Competitor X news announcements last 7 days")
OBSERVE: [Returns 12 results]

ACT: → web_search("Competitor X product pricing changes 2024")
OBSERVE: [Returns 4 relevant results]

ACT: → news_api(company="Competitor X", days=7)
OBSERVE: [Returns 3 press releases]

THINK: "I have sufficient information. Structuring the summary now."

OUTPUT (returned to Orchestrator):
{
  "competitor": "X",
  "news_summary": "Competitor X announced a Series C funding round of $85M on 
                   March 10. CEO stated expansion into enterprise segment.",
  "product_changes": "Launched new API tier on March 8. Pricing reduced 15% 
                      for mid-market accounts.",
  "sources": ["TechCrunch March 10", "Competitor X blog March 8"]
}
```

*[Identical process runs simultaneously for Competitors Y and Z]*

#### Step 3: Orchestrator Routes Research Output to Analysis Agent

```
[Orchestrator]
OBSERVE: "Research outputs received for X, Y, and Z."
THINK:   "All three research tasks are complete. 
          Passing combined output to Analysis Agent."

→ Analysis Agent receives: combined research summaries for X, Y, Z
                           + internal market position data
                           + last week's report for comparison
```

#### Step 4: Analysis Agent Processes and Interprets

```
[Analysis Agent]

OBSERVE: "Three competitor summaries received. Internal data available."

THINK: "Competitor X's funding and enterprise pivot is the most significant 
        development. Combined with Y's pricing changes, this suggests 
        increasing pressure on our mid-market positioning.
        I need to quantify the pricing gap before drawing conclusions."

ACT: → internal_data_query("our mid-market pricing vs. Competitor X and Y")
OBSERVE: [Returns pricing comparison table]

ACT: → calculator("Competitor X new price / our equivalent tier price")
OBSERVE: "Competitor X is now 18% cheaper than us in mid-market tier."

THINK: "This is actionable. The strategic implication is clear: 
        our mid-market pricing is now significantly above X and Y. 
        This is the primary recommendation for the executive summary."

OUTPUT (returned to Orchestrator):
{
  "key_finding": "X and Y have both reduced mid-market pricing in the past 7 days.
                  Combined effect: our mid-market tier is now 15-18% above 
                  nearest competitor.",
  "strategic_implication": "Increased churn risk in mid-market segment. 
                             Pricing review recommended before Q2.",
  "supporting_data": [pricing comparison table, funding announcement context]
}
```

#### Step 5: Orchestrator Routes to Writing Agent

```
[Orchestrator]
OBSERVE: "Analysis complete. Routing to Writing Agent with full context."

→ Writing Agent receives: research summaries + analysis output + report template
```

#### Step 6: Writing Agent Produces Final Report

```
[Writing Agent]

OBSERVE: "Full context received. Template specifies: executive summary first, 
          then findings by competitor, then recommendations."

THINK:   "The pricing story is the lead. I'll structure the executive summary 
          around that finding, then support it with the per-competitor details."

ACT: → document_create(template="ci_report_standard")
ACT: → document_write(section="executive_summary", 
                      content=[synthesized from analysis output])
ACT: → document_write(section="competitor_x", content=[from research + analysis])
... [continues for Y and Z]
ACT: → document_write(section="recommendations", content=[from analysis output])

OUTPUT: Completed report document → returned to Orchestrator
```

#### Step 7: Orchestrator Delivers Final Output

```
[Orchestrator]
OBSERVE: "Report complete."
THINK:   "Verify completeness against template requirements. 
          All sections present. Word count within range. 
          Delivering to requestor."

ACT: → send_report(to="bd_team@company.com", document=completed_report)

OUTPUT: "Weekly CI report delivered. 
         Key finding: mid-market pricing gap of 15-18% vs. X and Y. 
         Recommend pricing review before Q2."
```

### What This Simulation Reveals

Walking through this workflow step-by-step exposes the design decisions that determine whether a multi-agent system works in production:

- **Context isolation is deliberate:** Each agent receives only what it needs. The Research Agent does not see the analysis prompt; the Writing Agent does not query live data. This keeps each agent's reasoning focused and its context window manageable.
- **The orchestrator does not execute:** It only reasons about delegation and synthesis. Keeping execution in the worker agents preserves a clean separation of concerns and makes each agent independently testable.
- **Parallelism is a design choice, not automatic:** The three research calls are parallel because the orchestrator explicitly delegates them simultaneously. This requires the platform or runtime to support concurrent agent execution.
- **The handoff format matters:** The structured JSON outputs from the Research and Analysis Agents make the downstream agents' work deterministic. Unstructured outputs would force the Analysis and Writing Agents to parse ambiguous text — introducing error surface.

---

## 3. Risks, Failures, and Coordination Mechanisms (10 min)

### Multi-Agent Systems Fail Differently

Single-agent failures are relatively straightforward: the agent reasons incorrectly, calls the wrong tool, or produces a bad output. Multi-agent failures are more complex because they can emerge from the interaction between agents, not just from any individual agent's behavior.

<img width="931" height="391" alt="image" src="https://git.generalassemb.ly/user-attachments/assets/76aaba06-03ef-4178-9f95-38104db0271b" />


### The Coordination Design Checklist

Before deploying a multi-agent system:

| Design Decision | Defined? |
|---|---|
| Coordination model selected and justified | <div align="center">☐</div> |
| Agent roles and responsibilities clearly separated | <div align="center">☐</div> |
| Handoff format standardized across all agent pairs | <div align="center">☐</div> |
| Output validation gates defined for high-risk transitions | <div align="center">☐</div> |
| Conflict resolution protocol defined | <div align="center">☐</div> |
| Circular dependency check completed | <div align="center">☐</div> |
| Timeout and fallback policies defined for each inter-agent communication | <div align="center">☐</div> |
| Injection defenses applied at every agent input boundary | <div align="center">☐</div> |
| Observability configured: full interaction trace across all agents | <div align="center">☐</div> |

---

## 4. Break (10 min)

---

## 5. Lab (60 min)

*Lab details in the Lab Guide document.*

---

## 6. Wrap-Up & Reflection (5 min)



> Next session: **Integration with Business Data** 

---

## Resources

- [CrewAI — Multi-Agent Coordination Documentation](https://docs.crewai.com/concepts/crews)
- [LangGraph — Multi-Agent Workflows](https://langchain-ai.github.io/langgraph/tutorials/multi_agent/multi_agent_collaboration/)
- [AutoGen — Microsoft Multi-Agent Framework](https://microsoft.github.io/autogen/)
- Chase, H. — *LangChain Blog: Multi-Agent Systems* (2024)
- Lilian Weng — *LLM-powered Autonomous Agents: Planning* (2023) — [lilianweng.github.io](https://lilianweng.github.io/posts/2023-06-23-agent/#planning)
- OWASP — *Top 10 for LLMs: Insecure Plugin Design and Excessive Agency* (2023)
