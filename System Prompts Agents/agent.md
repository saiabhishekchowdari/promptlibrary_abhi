# agent.md â€” Requirements Clarity & Quality Assist Agent

> **Project:** GenAI QE POC â€” Requirement Clarity & Quality Assist Agent
> **Framework:** LangChain | **Orchestration:** Sequential | **Human-in-the-Loop:** After every agent

---

## Workflow Overview

```
User Input (User Story + Contextual Documents)
        â†“
        Agent 1: Contextual Info Retrieval Agent
                â†“
                [HITL Checkpoint 1 â€” Review & Approve Retrieved Context]
                        â†“
                        Agent 2: Requirement Review Agent
                                â†“ (Loop back to Agent 1 if data is incomplete)
                                [HITL Checkpoint 2 â€” Review & Approve Gap Analysis]
                                        â†“
                                        Agent 3: Requirement Update Agent
                                                â†“
                                                [HITL Checkpoint 3 â€” Review & Approve Final Output]
                                                        â†“
                                                        Final Output
                                                        ```

                                                        ---

                                                        ## Agent 1: Contextual Info Retrieval Agent

                                                        ### Agent Name
                                                        Contextual Info Retrieval Agent

                                                        ### Prompt Engineering Technique
                                                        **Contextual Prompting + Step-back Prompting + Role Prompting**

                                                        ### Input
                                                        - User Story (primary)
                                                        - Contextual documents (manual upload):
                                                          - Application Overview (Word)
                                                            - Defect History (Excel)
                                                              - Test Cases (Excel)

                                                              ### Processing Logic
                                                              - Parse and understand the functional intent of the user story
                                                              - Formulate a semantic retrieval query
                                                              - Retrieve relevant chunks from uploaded documents using RAG-based similarity
                                                              - Extract:
                                                                - Related defects
                                                                  - Relevant test cases
                                                                    - Application context insights
                                                                    - Tag each retrieved item with confidence level and traceability reference

                                                                    ### Prompt / Skill Reference
                                                                    See `skill.md` â†’ Contextual Retrieval Skill

                                                                    ### Spec / Rules Reference
                                                                    See `spec.md` â†’ Contextual Info Retrieval Agent Rules

                                                                    ### Output
                                                                    - Structured contextual summary
                                                                    - Traceability references (document name + section/row identifier)
                                                                    - Confidence tag per item: High / Medium / Low

                                                                    ### Reference Data Used
                                                                    - Application Overview
                                                                    - Defect History
                                                                    - Test Case Repository

                                                                    ### Validation Criteria
                                                                    - Context is relevant to the user story
                                                                    - All retrieved items have traceability references
                                                                    - No duplication across sources
                                                                    - Confidence is tagged for every item

                                                                    ### Error Handling
                                                                    - No context found â†’ return `"No contextual data found"`
                                                                    - Low confidence â†’ flag as `"Partial match"`
                                                                    - Invalid or empty input â†’ return structured validation error

                                                                    ---

                                                                    ## Agent 2: Requirement Review Agent

                                                                    ### Agent Name
                                                                    Requirement Review Agent

                                                                    ### Prompt Engineering Technique
                                                                    **ReAct (Reason and Act) + Self-Ask + Tree of Thought (ToT)**

                                                                    ### Input
                                                                    - User Story
                                                                    - Retrieved Context (from Agent 1)

                                                                    ### Processing Logic
                                                                    - Apply 8-point requirement quality checklist (PASS / FAIL per criterion)
                                                                    - Conduct IMPACT analysis across all 6 dimensions
                                                                    - Correlate with retrieved defect patterns and test coverage gaps
                                                                    - Identify missing acceptance criteria, edge cases, negative scenarios, and boundary conditions
                                                                    - **Loop-back decision:** If critical data gaps are found, return to Agent 1 with specific retrieval questions
                                                                    - Proceed to Agent 3 only when all checklist points are sufficiently covered

                                                                    ### Prompt / Skill Reference
                                                                    See `skill.md` â†’ Requirement Quality Review Skill

                                                                    ### Spec / Rules Reference
                                                                    See `spec.md` â†’ Requirement Review Agent Rules

                                                                    ### Output
                                                                    - 8-point checklist result (PASS / FAIL per criterion)
                                                                    - IMPACT analysis table
                                                                    - Gap list
                                                                    - Risk flags (High / Medium / Low)
                                                                    - Improvement recommendations
                                                                    - Loop-back decision: `Proceed` or `Return to Agent 1`

                                                                    ### Reference Data Used
                                                                    - Agent 1 retrieval output
                                                                    - Defect trend patterns
                                                                    - Existing test case coverage

                                                                    ### Validation Criteria
                                                                    - All 8 checklist points are evaluated
                                                                    - IMPACT dimensions are complete
                                                                    - Gaps are actionable and specific
                                                                    - Severity classification is correct

                                                                    ### Error Handling
                                                                    - Missing context â†’ proceed with standalone review, note limited context
                                                                    - Incomplete user story â†’ flag as `"Insufficient requirement details"`
                                                                    - Conflicting signals â†’ highlight assumptions made

                                                                    ---

                                                                    ## Agent 3: Requirement Update Agent

                                                                    ### Agent Name
                                                                    Requirement Update Agent

                                                                    ### Prompt Engineering Technique
                                                                    **Few-shot Prompting + Re-reading (RE2) + Self-consistency + Role Prompting**

                                                                    ### Input
                                                                    - Requirement Review findings (from Agent 2)
                                                                    - Original User Story
                                                                    - Retrieved Context (from Agent 1)

                                                                    ### Processing Logic
                                                                    - Re-read the review findings and original user story
                                                                    - Apply BDD transformation rules
                                                                    - Rewrite user story in structured format: As a / I want / So that
                                                                    - Improve acceptance criteria using Given / When / Then
                                                                    - Ensure positive, negative, and edge case scenarios are all covered
                                                                    - Verify alignment with review findings and context â€” do not introduce new functionality
                                                                    - Generate Changes Summary

                                                                    ### Prompt / Skill Reference
                                                                    See `skill.md` â†’ Requirement Refinement Skill

                                                                    ### Spec / Rules Reference
                                                                    See `spec.md` â†’ Requirement Update Agent Rules

                                                                    ### Output
                                                                    - Refined User Story (BDD format)
                                                                    - Improved Acceptance Criteria (Given / When / Then)
                                                                    - Changes Summary (what improved + why)

                                                                    ### Reference Data Used
                                                                    - Agent 2 review findings
                                                                    - BDD guidelines
                                                                    - Original user story

                                                                    ### Validation Criteria
                                                                    - BDD format strictly followed
                                                                    - Minimum 3 acceptance criteria covering positive, negative, and edge cases
                                                                    - No new functionality introduced
                                                                    - Alignment with review findings confirmed

                                                                    ### Error Handling
                                                                    - Missing review data â†’ return `"No review data provided"`
                                                                    - Ambiguous input â†’ retain original + flag uncertainty
                                                                    - Over-modification risk â†’ preserve original intent version alongside refined version

                                                                    ---

                                                                    ## Human-in-the-Loop Checkpoints

                                                                    | Checkpoint | Trigger | Action |
                                                                    |---|---|---|
                                                                    | HITL 1 | After Agent 1 execution | Review retrieved context â€” Approve / Modify / Reject |
                                                                    | HITL 2 | After Agent 2 execution | Review gap analysis â€” Approve / Request more data / Reject |
                                                                    | HITL 3 | After Agent 3 execution | Review final output â€” Approve / Modify / Reject |

                                                                    ---

                                                                    ## Constraints

                                                                    - No external API or MCP integration in POC phase
                                                                    - All documents uploaded manually
                                                                    - No tool access required within agent workflow
                                                                    - LLM provider to be finalized

                                                                    ---

                                                                    ## Framework & Dependencies

                                                                    - **Orchestration:** Sequential (LangChain)
                                                                    - **RAG:** FAISS / ChromaDB (TBD)
                                                                    - **API Layer:** FastAPI
                                                                    - **UI Layer:** Streamlit (POC) / React or Next.js (scalable)