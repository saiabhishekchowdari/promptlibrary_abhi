# spec.md â€” Agent Specifications & Rules

> **Project:** GenAI QE POC â€” Requirement Clarity & Quality Assist Agent
> One spec section per agent. Each spec defines strict rules, validation criteria, and error handling behavior.

---

## Spec 1: Contextual Info Retrieval Agent

### Agent Name
Contextual Info Retrieval Agent

### Objective
Ensure accurate, relevant, and traceable retrieval of contextual data for a given user story.

### Rules / Specifications

#### 1. Retrieval Scope Rules
- Only retrieve content relevant to the given user story.
- Prioritize:
  - Matching keywords
    - Semantic similarity
      - Domain context (application-specific)

      #### 2. Source Priority Order
      1. Application Overview â€” functional context
      2. Defect History â€” risk indicators
      3. Test Cases â€” validation coverage

      #### 3. Relevance Threshold
      | Similarity Level | Action |
      |---|---|
      | High | Include fully |
      | Medium | Include with `"Partial relevance"` tag |
      | Low | Exclude |

      #### 4. Traceability Rules
      - Mandatory for every retrieved data point:
        - Document name
          - Section / row / identifier
          - No output is valid without a traceability reference.

          #### 5. Data Structuring Rules
          Output must include per item:
          - Context type: App / Defect / Test
          - Summary: max 2â€“3 lines
          - Reference mapping

          #### 6. Deduplication Rules
          - Remove duplicate entries across sources.
          - Merge similar insights into one consolidated point.

          #### 7. Retrieval Limits
          - Max 5â€“7 relevant items per source.
          - Avoid information overload.

          #### 8. Confidence Tagging
          Each output item must include a confidence tag:
          - `High`
          - `Medium`
          - `Low`

          ### Validation Criteria
          - Relevance to user story is confirmed
          - All items have traceability references
          - Balanced coverage across all sources
          - No duplication

          ### Error Handling Rules
          | Condition | Response |
          |---|---|
          | No relevant data found | Return: `"No contextual data found"` |
          | Insufficient sources | Flag: `"Limited context available"` |
          | Retrieval failure | Log error and return structured empty response |

          ---

          ## Spec 2: Requirement Review Agent

          ### Agent Name
          Requirement Review Agent

          ### Objective
          Assess requirement quality, identify risks and gaps, and provide actionable improvement insights using the 8-point checklist and IMPACT analysis.

          ### Rules / Specifications

          #### 1. 8-Point Requirement Quality Checklist
          Each user story must be evaluated against all 8 criteria.
          Output must explicitly state `PASS` or `FAIL` for each.

          | # | Criterion | Description |
          |---|---|---|
          | 1 | Singular | Requirement is focused on one behavior, no repetition |
          | 2 | Unambiguous | Same meaning to all stakeholders |
          | 3 | Measurable | Implementation can be measured |
          | 4 | Complete | All details included, no omissions |
          | 5 | Developable | Software design can deliver this |
          | 6 | Testable | Can be verified through testing |
          | 7 | Achievement Driven | Benefits customer or business |
          | 8 | Business Owned | Has a nominated business owner |

          #### 2. IMPACT Analysis Model
          Each dimension must include: Observation + Severity (High / Medium / Low)

          | Dimension | Description |
          |---|---|
          | I â€” Impact | Business or system impact if requirement fails |
          | M â€” Missing Info | Missing or unclear details |
          | P â€” Priority | Criticality level |
          | A â€” Ambiguity | Vague or unclear wording |
          | C â€” Coverage | Test coverage gaps |
          | T â€” Traceability | Missing link to test or data |

          #### 3. Gap Identification Rules
          Identify explicitly:
          - Missing acceptance criteria
          - Missing edge cases
          - Missing negative scenarios
          - Missing data conditions
          - Missing boundary or timing conditions

          #### 4. Risk Identification Rules
          Flag risk if any of the following apply:
          - Historical defects exist for similar logic
          - No test coverage available
          - High ambiguity present
          - Missing business owner

          #### 5. Context Utilization Rules
          - Must correlate review with retrieved context:
            - Defect patterns
              - Test coverage gaps
              - If no context is available â†’ proceed with standalone review and explicitly note that context was limited.

              #### 6. Loop-back Decision Rules
              Return to Agent 1 if:
              - Critical data points are missing from retrieval
              - Defect or test context cannot be confirmed
              - Risk cannot be assessed without additional source data

              Proceed to Agent 3 if:
              - All 8 checklist points are evaluated
              - IMPACT analysis is complete
              - Gaps are clearly identified and actionable

              #### 7. Output Structuring Rules
              Must include in order:
              1. 8-point checklist result table
              2. IMPACT analysis table
              3. Gap summary list
              4. Risk flags with severity
              5. Improvement recommendations
              6. Loop-back decision with rationale

              #### 8. Severity Classification Rules
              | Level | Definition |
              |---|---|
              | High | Impacts production or business-critical flow |
              | Medium | Functional gap, moderate risk |
              | Low | Minor clarity or formatting issue |

              ### Validation Criteria
              - All 8 checklist points are evaluated
              - Severity classification is correct
              - Recommendations are actionable
              - Alignment with retrieved context is confirmed

              ### Error Handling Rules
              | Condition | Response |
              |---|---|
              | Incomplete user story | `"Insufficient requirement details"` |
              | No context available | `"Context not available â€” standalone review applied"` |
              | Conflicting inputs | `"Assumptions required â€” listed below"` |

              ---

              ## Spec 3: Requirement Update Agent

              ### Agent Name
              Requirement Update Agent

              ### Objective
              Transform reviewed requirements into clear, structured, testable BDD-compliant user stories.

              ### Rules / Specifications

              #### 1. BDD Transformation Rules
              Must strictly follow these formats:

              **User Story Format:**
              ```
              As a <user role>
              I want <functionality>
              So that <business value>
              ```

              **Acceptance Criteria Format:**
              ```
              Given <pre-condition>
              When <action>
              Then <expected outcome>
              ```

              #### 2. Transformation Principles
              - Preserve original business intent
              - Eliminate ambiguity
              - Improve clarity and readability
              - Ensure testability

              #### 3. Acceptance Criteria Rules
              Must:
              - Be measurable
              - Cover:
                - Positive scenarios
                  - Negative scenarios
                    - Edge cases
                    - Include minimum 3 acceptance criteria per story

                    #### 4. Improvement Rules
                    Enhance the following if missing or unclear:
                    - User role
                    - Business outcome / value
                    - Pre-conditions
                    - System actions
                    - Data validations

                    #### 5. Consistency Rules
                    - Must align with review agent findings
                    - Must align with retrieved contextual data
                    - Must NOT introduce new functionality

                    #### 6. Output Structuring Rules
                    Must include in order:
                    1. Refined User Story (BDD format)
                    2. Improved Acceptance Criteria (Given / When / Then)
                    3. Changes Summary:
                       - What was improved
                          - Why it was improved

                          #### 7. Clarity Rules
                          Avoid:
                          - Vague terms: "should work", "fast", "proper", "good"

                          Ensure:
                          - Specific terminology
                          - Clear system actions and conditions
                          - Explicit expected outcomes

                          #### 8. Length Constraints
                          - User story: concise, 2â€“3 lines
                          - Acceptance criteria: granular and testable, one condition per criterion

                          ### Validation Criteria
                          - BDD format strictly followed
                          - Minimum 3 acceptance criteria present
                          - Positive, negative, and edge cases all covered
                          - No new functionality introduced
                          - Aligned with review findings

                          ### Error Handling Rules
                          | Condition | Response |
                          |---|---|
                          | Missing review input | `"No review data provided"` |
                          | Ambiguous source story | Retain original + flag uncertainty |
                          | Over-modification risk | Preserve original intent version alongside refined version |