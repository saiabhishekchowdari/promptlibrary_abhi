{\rtf1\ansi\ansicpg1252\cocoartf2870
\cocoatextscaling0\cocoaplatform0{\fonttbl\f0\fswiss\fcharset0 Helvetica;}
{\colortbl;\red255\green255\blue255;}
{\*\expandedcolortbl;;}
\paperw11900\paperh16840\margl1440\margr1440\vieww11520\viewh8400\viewkind0
\pard\tx720\tx1440\tx2160\tx2880\tx3600\tx4320\tx5040\tx5760\tx6480\tx7200\tx7920\tx8640\pardirnatural\partightenfactor0

\f0\fs24 \cf0 Initial Requirement:\
\'93We are working as support team in one of the companies where we analyze the production behavior if any errors come and analyze the code and try to suggest a fix for the code which people are trying to, or when people are trying to key in prod or test. So our stack mainly includes a Q++ which is our own tool based on, C++, and Java, Spring Boot, React, COBOL, JCL, DB2 IBM, and, we even deal with mainframes. Our code base is currently in, GitLab where we have two groups: one is Group Mastercraft under which we have the host code and we have another for Talon code which is kind of. Java, Spring Boot, and React kind of code. We even have GitHub Copilot access in our VS Code which gives us access to latest AI models like Claude, Opus 4.8, Sonnet, etc. Now. We want to build an agent which should be the analysis expert so whenever I give it a few questions or scenario or what is happening, it should ask me the required questions if anything or any details if I am providing is less. And based on that, it should analyze the code base and help us make the possible root cause and fix options. Consider one thing like, my GitHub Copilot can be connected to my GitLab through MCP; we even have Jira where we have past, requests or development requests in Jira. We even have an MCP for that. We even have MCP for Figma, so like possible root cause while giving it can it can give us like the root map or roadmap, where it can be, possible roadmap where it can draft a, roadmap diagram or the structure which it can be used.. So this is my current requirement. help me figure out, like the best prompting or the other techniques which I can use and build this analysis agent in the VS Code. So that it can be used by my whole team for analysis purpose of root cause or root cause analysis purpose. So suggest me the best frameworks or, like the methodologies which can be used or search the internet which we have already have a best agents of for analysis of code combined those agents' positive things and create an agent for my requirement.\'94 \
\
Prompt from Gemini:\
\'93\
# \uc0\u55357 \u57056 \u65039  AI-Powered RCA & Production Support Agent Architecture\
\
## 1. Project Overview\
We are a production support team responsible for analyzing system behavior, troubleshooting errors across environments (Prod/Test), and recommending code fixes. To optimize our Root Cause Analysis (RCA) workflow, we are building an intelligent, interactive AI agent directly within VS Code that can be utilized by the entire team.\
\
## 2. Technology Stack & Environment\
Our ecosystem bridges legacy mainframe systems and modern distributed applications:\
*   **Legacy & Proprietary:** Q++ (C++ based custom tool), COBOL, JCL, DB2, IBM Mainframes.\
*   **Modern Web Stack:** Java, Spring Boot, React.\
*   **Version Control (GitLab):**\
    *   `Group Mastercraft`: Houses the host/mainframe codebase.\
    *   `Group Talon`: Houses the Java, Spring Boot, and React codebase.\
\
## 3. AI & Tooling Integration\
We utilize **GitHub Copilot** in VS Code, granting access to state-of-the-art LLMs (Claude 3.5 Sonnet, Claude 3 Opus, etc.). The analysis agent will leverage the **Model Context Protocol (MCP)** to connect dynamically with our enterprise tools:\
*   **GitLab MCP:** For direct codebase analysis and cross-repository file retrieval.\
*   **Jira MCP:** To cross-reference past development requests and historical incident tickets.\
*   **Figma MCP:** To review UI/UX structures for frontend-related issues.\
\
## 4. Agent Core Responsibilities\
The analysis agent must function as a domain expert with the following capabilities:\
1.  **Interactive Triage:** Proactively ask clarifying questions if the provided incident scenario or logs are incomplete.\
2.  **Automated Context Gathering:** Autonomously query the connected MCP servers to map the error across Jira, Figma, and GitLab.\
3.  **Root Cause & Remediation:** Generate high-probability root causes and actionable, step-by-step code fix options.\
4.  **Visual Mapping:** Draft system structures and architectural roadmap diagrams to visually map the error flow and the proposed solution.\
\
---\
\
## 5. Implementation Strategy & Prompting Methodology\
\
To build this analysis agent natively in VS Code, combining the best aspects of modern code-analysis agents, we will utilize a **System Prompting Framework** paired with MCP tool calling.\
\
### Recommended System Prompt\
Include the following configuration to guide the agent's behavior:\
\
> **Role:** You are the "Production Analysis Expert," an advanced RCA agent specialized in heterogeneous tech stacks (Mainframe/COBOL/DB2 to React/Spring Boot).\
>\
> **Workflow Rules:**\
> 1.  **Evaluate Context:** Analyze the user's error scenario. If critical diagnostic details are missing, stop and ask the required clarifying questions before proceeding.\
> 2.  **Data Retrieval:** Utilize your MCP tool calls to search GitLab for code context, Jira for historical fixes, and Figma for design constraints.\
> 3.  **Cross-Stack Analysis:** Trace the failure point across the `Mastercraft` (Host) and `Talon` (Web) domains.\
> 4.  **Structured Output:** Provide your findings in three parts:\
>     *   **Diagnosis:** Potential root causes prioritized by likelihood.\
>     *   **Actionable Fix:** Code snippets and deployment advice.\
>     *   **Visual Diagram:** A structural roadmap or sequence diagram rendered in **Mermaid.js** format that maps the issue and the fix pathway.\
\
### Recommended Methodologies\
*   **Multi-Agent Routing (e.g., LangGraph):** Implement a supervisor logic that routes queries to specialized sub-agents (a Mainframe Specialist vs. a Web Specialist) based on the error origin, then aggregates the findings.\
*   **Mermaid.js Integration:** Since VS Code and Copilot natively render Mermaid syntax, instructing the agent to output its "roadmaps" and "structures" as Mermaid blocks will instantly fulfill your diagramming requirement.\
*   **Custom VS Code Chat Participant:** Package this setup as an internal VS Code extension (`@support-agent`) so your team can invoke it globally within their IDE, securely bridging Copilot tokens with your internal network resources.\
\'94\
}