## 1. What is the purpose of this phase?

Before writing code, the AI FDE first needs to understand **what solution should actually be built**.

The video emphasizes that this stage is not primarily about coding. It is a **brainstorming and ideation phase** where we identify:

* What data the client has
* What type of data it is
* What users want to ask
* How the AI should access that data
* What external knowledge/policies are available
* How security should be handled
* What architecture/approach should be used

The central requirement from the client is essentially:

> **“Talk to our data.”**

Everything in the solution discussion is designed around making that possible. 

---

# 2. What has already been provided by the client?

Before designing the solution, assume that two important things are already available:

### 1. Legacy database

The legacy system/database has already been created and is treated as something **given by the client**.

The AI FDE doesn't need to build this database from scratch.

### 2. SOP / Policy documents

The client also provides **Standard Operating Procedures (SOPs)**.

An SOP tells employees what action they should take when a particular situation occurs.

For example:

```text
Temperature > 4°C
        ↓
Contact driver
        ↓
Start auxiliary cooling
        ↓
If ETA delay > 1 hour
        ↓
Divert vehicle to emergency cold storage
```

So the AI assistant must eventually be able to understand these policies and use them when answering questions. 

---

# 3. Example: Cold Chain Logistics SOP

The project is based around **cold-chain logistics**.

Imagine refrigerated vehicles transporting perishable products.

The system has IoT sensors that monitor things such as:

* Temperature
* Location
* Coordinates
* ETA
* Shipment information
* Risk
* Weather-related conditions

The example SOP specifies that fresh perishables should remain between:

**0°C and 4°C**

If the temperature goes above 4°C, it becomes a critical breach.

The SOP then tells the operator what to do.

For example:

### Temperature breach

If:

```text
Temperature > 4°C
```

then:

1. Dispatcher contacts the driver.
2. Driver starts auxiliary cooling.
3. If ETA delay exceeds one hour:

   * Divert the vehicle to the nearest emergency cold storage.

This information isn't necessarily present in the database itself.

It exists in the **company's internal SOP/policy documentation**. 

---

# 4. Another SOP example: Route congestion

The video gives another hypothetical SOP around port congestion.

Suppose:

```text
Severity Index > 7
```

Then:

```text
Do not hold freight at the port
        ↓
Divert active shipments
```

Again, this isn't simply a database query.

The system needs to know the company's **business rules**.

---

# 5. Risk classification

Another example is:

```text
Shipment = High Risk
        +
Delay Probability
        ↓
Escalate to Logistics Manager
```

Normally, a human employee would:

1. Look at the data.
2. Read the SOP.
3. Determine what situation applies.
4. Follow the prescribed procedure.
5. Contact the appropriate person.

The goal of the AI assistant is to automate much of this interaction.

For example, instead of the employee manually reading the SOP, they could ask:

> "What should I do with this shipment?"

The AI can combine:

**Database information + SOP knowledge**

and explain the appropriate procedure. 

---

# 6. The first major question: Is the data structured?

This is one of the most important decisions in the entire solution design.

Ask:

> **Is our data structured?**

There are two broad possibilities.

### Structured data

Usually:

```text
Relational Database
        ↓
Rows + Columns
```

For example:

| Shipment | Temperature | Location | Risk |
| -------- | ----------: | -------- | ---- |
| S001     |         3°C | SF       | Low  |
| S002     |         7°C | LA       | High |

### Unstructured data

Examples:

* Text
* PDF
* Images
* Documents

The two types require **different approaches**. 

---

# 7. Why RAG isn't suitable for the relational database

Suppose the relational database contains approximately:

```text
30,000 rows
```

You cannot simply take all 30,000 rows and put them into an LLM's context.

Conceptually:

```text
30,000 database rows
        ↓
      LLM
```

This isn't a practical approach.

Instead, the AI needs a mechanism to **retrieve only the relevant data**.

Therefore, structured relational data requires a different solution.

The video considers APIs, tools, MCP, and Text-to-SQL for this part. 

---

# 8. What about APIs?

If the company already has APIs for analytics, those APIs can be used.

For example:

```text
AI Assistant
      ↓
Analytics API
      ↓
Database
      ↓
Result
      ↓
AI Assistant
```

An API might already provide information such as:

* Shipment count per month
* Shipment count per week
* Rolling four-week shipment count
* Other business analytics

But the important question is:

> **What if the required API doesn't exist?**

That's where the solution design needs another approach. 

---

# 9. In this project, we have BOTH types of data

This is important.

The project doesn't have only structured data.

It has:

### Structured

```text
Relational Database
```

### Unstructured

```text
SOP / Policy documents
```

Therefore:

```text
                    DATA
                     |
          ┌──────────┴──────────┐
          ↓                     ↓
   Structured Data       Unstructured Data
          ↓                     ↓
 Relational Database       SOP / Documents
          ↓                     ↓
    Text-to-SQL                 RAG
```

The solution must handle **both paths**. 

---

# 10. What questions will users ask?

Before designing tools, we need to understand the expected queries.

The video identifies several categories.

## A. Location-based queries

Example:

> "Show me shipment analytics for San Francisco."

The system should be able to filter/query based on location.

---

## B. Coordinate-based queries

Because shipments contain coordinates, users may ask:

> "How many high-risk shipments are currently within this area?"

The system therefore needs to understand geographic/coordinate information.

---

## C. Perishable shipments

Users might ask:

> "Which shipments contain perishable goods?"

or:

> "Which perishable shipments are currently at risk?"

---

## D. Temperature queries

Examples:

> "Which shipments have exceeded the temperature threshold?"

or:

> "Which vehicles are currently experiencing temperature issues?"

---

## E. Refrigeration queries

The temperature data can also be used to investigate whether the vehicle's refrigeration system is functioning correctly.

---

## F. Heat-spike queries

The system may encounter unusual temperature spikes.

The user could ask:

> "Was this temperature spike expected?"

or:

> "Is the refrigeration system working correctly during this heat spike?"

---

## G. Weather-related queries

The system could potentially combine:

```text
Shipment
+
Location
+
Weather
+
Temperature
```

For example, weather conditions could be considered alongside shipment temperature.

---

## H. SOP queries

Users could ask:

> "What SOPs does the company follow?"

or:

> "What should I do when the shipment temperature exceeds 4°C?"

This requires access to the company's SOP knowledge. 

---

# 11. Why SOP querying is important

Different employees may need SOP information.

### New employee

A new employee may ask:

> "What procedures do we follow for temperature breaches?"

### Manager

A manager may need to quickly check a policy that has recently changed.

The problem is that employees cannot realistically remember every detail of every SOP.

Therefore:

```text
Employee
   ↓
AI Assistant
   ↓
SOP Knowledge
   ↓
Answer
```

becomes useful. 

---

# 12. Security becomes a major concern

Once an AI agent can interact with a database, another important question appears:

> **What prevents the agent from modifying or deleting data?**

This is critical.

Imagine the agent has database access.

A malicious or incorrect generated command could potentially attempt:

```sql
DELETE FROM shipments;
```

or:

```sql
INSERT ...
```

That is obviously dangerous.

Therefore, the solution needs **database-level access restrictions**. 

---

# 13. Principle: Don't give the AI admin credentials

The video's approach is:

### Create a dedicated database user.

For example:

```text
Database
   |
   ├── Admin User
   ├── Developer User
   └── AI Agent User
```

The AI agent receives credentials for the **AI Agent User**, not an administrator.

That user should have only the permissions required for its job.

For example:

```text
AI Agent User
      ↓
SELECT
      ✓

INSERT
      ✗

DELETE
      ✗
```

So even if the agent attempts to execute a destructive operation, the database itself rejects it.

This creates a security boundary **outside the LLM**. 

---

# 14. Data sources

At this point, the project has two major sources of knowledge:

```text
                    AI Assistant
                         |
            ┌────────────┴────────────┐
            ↓                         ↓
    Relational Database           SOP Knowledge
            |                         |
      Structured Data           Business Rules
```

The relational database contains actual operational data.

The SOP contains the **knowledge/rules for interpreting and responding to situations**. 

---

# 15. Structured data solution — MCP

One possible solution considered is **MCP**.

The idea is to expose database functionality through tools.

Conceptually:

```text
User
 ↓
AI Agent
 ↓
MCP
 ↓
Custom Tools
 ↓
Database
 ↓
Result
 ↓
AI Agent
 ↓
User
```

The engineer creates tools that the agent can invoke.

The problem becomes:

> **How many tools do we need?**

For example:

```text
get_shipments()
get_shipments_by_location()
get_high_risk_shipments()
get_temperature_breaches()
...
```

The exact number depends heavily on what users need to ask.

Therefore, understanding the client's expected queries is extremely important. 

---

# 16. Alternative: Text-to-SQL

The second major solution is:

> **Text-to-SQL**

This is a very important concept.

Instead of manually creating a tool for every possible query:

```text
User question
      ↓
AI Agent
      ↓
Generate SQL
      ↓
Database
      ↓
SQL Result
      ↓
AI Agent
      ↓
Answer
```

### Example

User asks:

> "How many high-risk shipments are currently in San Francisco?"

The AI could generate something conceptually like:

```sql
SELECT COUNT(*)
FROM shipments
WHERE location = 'San Francisco'
AND risk = 'High';
```

The generated SQL is then executed against the database.

The result goes back to the AI. 

---

# 17. Problem with Text-to-SQL

The biggest issue is:

> **The generated SQL might be wrong.**

If the SQL is incorrect:

```text
User question
      ↓
AI
      ↓
Incorrect SQL
      ↓
Database
      ↓
Error / Wrong result
```

The system may not get the desired result. 

---

# 18. Advantage of Text-to-SQL

The major advantage is that it is **not limited by a fixed number of tools**.

With MCP:

```text
20 business questions
↓
Potentially many tools
```

If tomorrow the business introduces five additional types of queries, new tools may need to be created.

With Text-to-SQL, you primarily provide the AI with the relevant database schema/context.

If new columns are added, the schema/context can be updated.

The AI can then generate SQL dynamically.

Therefore:

### MCP/tool approach

```text
More functionality
        ↓
Potentially more tools
        ↓
More engineering
```

### Text-to-SQL

```text
More database capabilities
        ↓
Updated schema/context
        ↓
AI generates appropriate SQL
```

This makes Text-to-SQL more flexible for this particular scenario. 

---

# 19. Can Text-to-SQL recover from errors?

Yes, potentially.

The agent can use a **multi-turn process**.

For example:

```text
User Question
     ↓
Generate SQL
     ↓
Execute SQL
     ↓
Error
     ↓
Agent sees error
     ↓
Generate corrected SQL
     ↓
Execute again
     ↓
Result
```

The agent needs visibility into previous messages/results to do this.

This becomes a **multi-turn conversation** rather than a single request-response interaction. 

---

# 20. Which approach is selected?

For this particular project, the video chooses:

> **Text-to-SQL for the relational database.**

The reasons given include:

* More flexible
* Not limited to a predefined number of tools
* Faster to prototype for this relatively simple database
* Can support additional query types without creating a separate tool for each one

The video explicitly marks this path as the selected solution. 

---

# 21. Unstructured data solution — RAG

Now we move to the second data source:

> **SOP knowledge**

For this, the proposed approach is:

> **RAG — Retrieval-Augmented Generation**

The user asks a question.

The system searches the SOP knowledge base for relevant information.

Then the retrieved information is provided to the agent.

Conceptually:

```text
User Question
      ↓
AI Agent
      ↓
RAG Tool
      ↓
Semantic Search
      ↓
Relevant SOP Context
      ↓
AI Agent
      ↓
Final Answer
```

The important point is that **RAG is treated as a tool that the agent can use**. 

---

# 22. Why semantic search?

The system shouldn't simply search for exact matching words.

For example, suppose the SOP says:

> "If temperature exceeds the acceptable threshold, initiate auxiliary cooling."

The user might ask:

> "What should I do if the refrigerated truck gets too hot?"

The wording is completely different.

A simple keyword/fuzzy search may not be sufficient.

Therefore, the system needs:

> **Semantic search**

Semantic search attempts to retrieve information based on **meaning**, rather than simply matching exact words.

RAG provides this capability. 

---

# 23. RAG requires a chunking strategy

Documents cannot necessarily be placed into a vector database as one giant block.

They need to be divided into smaller pieces called:

> **Chunks**

Example:

```text
SOP Document
      ↓
 ┌────┴─────┐
 ↓          ↓
Chunk 1    Chunk 2
 ↓          ↓
Chunk 3    Chunk 4
```

The way these chunks are created is called the:

> **Chunking strategy**

The chunking strategy affects retrieval quality. 

---

# 24. SOP modification and deletion

Another important consideration:

> **SOPs can change.**

Suppose today's SOP says:

```text
Temperature > 4°C
→ Start auxiliary cooling
```

Later the company changes it.

Now the AI must stop using the old rule.

Therefore, the RAG system needs a strategy for:

* Updating chunks
* Modifying existing information
* Removing obsolete information
* Keeping the vector database synchronized with the current SOP

This is called the **modification/deletion strategy** in the discussion. 

---

# 25. Source validation

This is one of the most important RAG design requirements.

The response should contain **source information**.

Why?

Because the AI shouldn't simply invent an answer.

The desired behavior is:

```text
User Question
      ↓
RAG Search
      ↓
Relevant source found?
      ↓
     YES
      ↓
Generate answer
```

But:

```text
Relevant source found?
      ↓
      NO
      ↓
"I couldn't find this information."
```

Instead of:

```text
No source
   ↓
LLM guesses
   ↓
Hallucinated answer
```

So the response should be grounded in retrieved source information. 

---

# 26. File ingestion logic

Another important design consideration is that different document types shouldn't necessarily be processed identically.

Possible sources include:

* Markdown (`.md`)
* PDF
* TXT
* XLSX
* CSV

For example:

### Markdown

Has:

```text
# Heading
## Subheading
### Section
```

So the structure can be used during ingestion.

### PDF

May contain:

* Text
* Tables
* Pages
* Headers
* Footers

### TXT

Generally has much less structural information.

### XLSX / CSV

Are tabular rather than normal textual documents.

Therefore, the system needs **different ingestion/injection logic depending on the source type**. 

---

# 27. Overall solution

The entire solution can now be summarized as:

```text
                         USER
                           |
                           ↓
                    AI ASSISTANT
                           |
              ┌────────────┴────────────┐
              ↓                         ↓
       Structured Data            SOP Knowledge
              ↓                         ↓
       Relational DB                  RAG
              ↓                         ↓
         Text-to-SQL              Semantic Search
              ↓                         ↓
         Query Result             SOP Context
              └────────────┬────────────┘
                           ↓
                       AI AGENT
                           ↓
                      Final Answer
```

And underneath the database layer:

```text
AI Agent
   ↓
Restricted DB User
   ↓
Database
```

so that the agent does not have unrestricted database permissions.

---

# 28. Final design decisions

The video ultimately arrives at these decisions:

| Problem                    | Selected approach                      |
| -------------------------- | -------------------------------------- |
| Relational/structured data | **Text-to-SQL**                        |
| SOP/document knowledge     | **RAG as a tool**                      |
| Semantic retrieval         | **RAG / semantic search**              |
| Database security          | **Restricted database user**           |
| SOP updates                | **Modification/deletion strategy**     |
| Hallucination prevention   | **Source validation**                  |
| Different file types       | **Separate ingestion/injection logic** |
| Agent interaction          | **Multi-turn capability**              |

The video explicitly summarizes the structured-data path as Text-to-SQL and the SOP path as RAG used as a tool. 

---

# 29. Why this whole discussion matters

The most important lesson isn't actually **Text-to-SQL vs RAG**.

The important lesson is:

> **Don't start coding before understanding the problem and designing the solution.**

The AI FDE first asks:

```text
What data do we have?
        ↓
What type of data is it?
        ↓
What questions will users ask?
        ↓
How can the agent access the data?
        ↓
What security restrictions are needed?
        ↓
How will documents be retrieved?
        ↓
How will documents be updated?
        ↓
How do we prevent hallucination?
        ↓
What solution should we build?
        ↓
HLD + LLD
        ↓
Implementation
```

The video specifically describes this as a **high-level solution design, not the coding stage**. 

---

# 30. What happens AFTER solution design?

Once the solution has been designed, the next step is documentation.

The video introduces:

### HLD — High-Level Design

Describes the system at a high level:

* Major components
* Overall architecture
* How components interact

### LLD — Low-Level Design

Goes into technical detail:

* Exact implementation approach
* Components
* Technical mechanisms
* How individual pieces will work

The purpose is to allow the engineering team to understand exactly what needs to be built before implementation begins. 

Then:

```text
Solution Discussion
        ↓
HLD + LLD
        ↓
Internal Technical Review
        ↓
Approval
        ↓
Business Pitch
        ↓
Actual Solution Building
```

This is the transition from **thinking about the solution → documenting the solution → building the solution**. 

---

## ⭐ The entire section in one picture

```text
                 CLIENT REQUIREMENT
                  "Talk to our data"
                          │
                          ↓
                 ┌─────────────────┐
                 │ Understand Data │
                 └────────┬────────┘
                          │
              ┌───────────┴───────────┐
              ↓                       ↓
       STRUCTURED DATA        UNSTRUCTURED DATA
       Relational DB             SOP / Docs
              │                       │
              ↓                       ↓
         Text-to-SQL                 RAG
              │                       │
              │                 Semantic Search
              │                       │
              │                 Chunking Strategy
              │                       │
              │              Update/Delete Strategy
              │                       │
              │                Source Validation
              │                       │
              └───────────┬───────────┘
                          ↓
                     AI AGENT
                          │
                 ┌────────┴────────┐
                 ↓                 ↓
             Security        Multi-turn Logic
                 │
                 ↓
        Restricted DB Access
                          │
                          ↓
                     USER ANSWER
                          │
                          ↓
                      HLD + LLD
                          │
                          ↓
                  Technical Review
                          │
                          ↓
                  Business Pitch
                          │
                          ↓
                    BUILD SYSTEM
```

**If you're learning AI FDE, this is probably the single most important mental model from this chapter:** first classify the data and understand the queries; then choose the retrieval/tool mechanism; then design security and update strategies; **only after that start implementation.**
