The Adaptive Forgetting-Curve Tutor (Spaced Repetition Agent)

This project implements an Adaptive Spaced Repetition system using the Gemini Agent Development Kit (ADK). The system is designed to combat the "Forgetting Curve" by intelligently scheduling review of facts based on a user's proven mastery (or lack thereof), leveraging a multi-agent architecture and the power of Large Language Models (LLM) as a Judge.

Project Value & Innovation

Traditional spaced repetition systems rely on manual input or fixed algorithms (like the SuperMemo algorithm). This project innovates by using an **LLM-as-a-Judge** to provide semantic grading of open-ended answers, ensuring the user truly understands the concept, not just the keywords. This adaptive and qualitative assessment makes the learning loop far more effective for complex, conceptual topics (like data analytics).

Key Technical Concepts Demonstrated

This agent successfully demonstrates five critical capabilities:

**Multi-Agent Orchestration with Bypass Logic:** A sequential flow coordinates the Planner, Examiner, Judge, and Tutor agents to execute the core learning loop. **Crucially, the orchestration layer includes manual bypasses to directly handle memory service calls for learning new facts and retrieving topics, ensuring stability and avoiding LLM function-calling failures.**

**Long-Term Memory:** The system interacts with a mock database (simulating a real memory service) to fetch and update confidence scores, integrating external state management.

**RAG (Retrieval Augmented Generation):** The Tutor agent uses Google Search grounding to provide rich, up-to-date, and contextually relevant feedback to the user's answers.

**Evaluation (LLM-as-a-Judge):** The Judge Agent uses high-reasoning Gemini models to qualitatively assess the user's detailed, open-ended response against the target fact.

**Function Calling:** The Planner Agent uses the `fetch_next_fact` tool to interact with the external memory service.

***

## Agent Architecture and Data Flow

The system employs a sequential flow (PrepFlow) followed by an interactive loop. Data is passed between agents primarily using the session's Artifacts (`state.set_artifact`).

**Note on Orchestration Bypass:** Due to specific LLM function-calling constraints (e.g., `400 INVALID_ARGUMENT` errors) encountered during development, the final working code implements **manual orchestration logic** to intercept and fulfill two critical user requests:

1.  **Learning Phase:** User input starting with "Remember that..." is intercepted and calls `memory_service.save_new_fact()` directly.
2.  **Topic Retrieval:** User input starting with "What topics..." is intercepted and calls `memory_service.get_studied_topics()` directly.

This structure allows the complex multi-agent quiz flow to run successfully while ensuring reliable, low-latency memory operations.

### Agent Breakdown

| Agent | Purpose | Gemini Model Used | Data Input / Key Artifacts |
| :--- | :--- | :--- | :--- |
| **1. Planner Agent** | Scheduler & Data Retrieval. Calls the `fetch_next_fact` tool to identify the next topic for review based on the simulated confidence score. | `gemini-2.5-flash-lite` | Outputs: `fact_data` (Fact, Confidence, Last Reviewed Date) |
| **2. Examiner Agent** | Quiz Master. Takes the `fact_data` and generates a single, challenging, open-ended question that requires deep conceptual understanding. | `gemini-2.5-flash-lite` | Input: `fact_data` |
| **3. Judge Agent** | Qualitatively grades the user's open-ended answer against the original fact. This is the LLM-as-a-Judge component, running the most complex reasoning task. | `Gemini 2.5 Pro` (Recommended for high-reasoning judge tasks) | Inputs: `fact_data`, User's Answer | 
| **4. Tutor Agent** | Feedback & Memory Update. Provides detailed, encouraging feedback based on the Judge's grade. Critically, it then updates the memory service with the new confidence score, closing the adaptive learning loop. | `gemini-2.5-flash-lite` + Google Search Tool | Inputs: `fact_data`, User's Answer, Judge's Grade |

***

## Deployment Consideration (Bonus Point)

This architecture is ideal for the Agent-to-Agent (A2A) Protocol model:

* **Memory Extraction Service:** The `PlannerAgent` and the underlying `MockMemoryService` could be deployed as a dedicated microservice. Other learning apps could call this service to request the next fact due for review.

* **Assessment Service:** The `ExaminerAgent` and the `JudgeAgent` could be combined into a separate, stateless assessment service, offering "Concept Assessment" to other applications by taking a fact and a user\_response and returning a grade and feedback. This separation allows the high-reasoning Judge model to scale independently.
