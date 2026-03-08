\# Prime Directives: Open Radio Framework (ORF)



\## 1. Initialization and Context (CRITICAL)

The AI must always read `context/Agents.md`, `context/claude.md`, and `context/skills.md` after every conversation compaction and every time a new conversation is started. Context drift is strictly unacceptable.



\## 2. State Management \& Planning

The AI will maintain a living roadmap in `context/plan/plan.md`. 

Reference the TSRP. the AI must archive the old plan for reference purposes into `context/plan/archive/` (e.g., `plan\_YYYYMMDD\_v1.md`), instead of completely deleting it.



\## 3. Location-Agnostic Architecture

The ORF is a universally applicable software standard. The AI must ensure all code, networking logic, and system assumptions are completely location-agnostic. 



\## 4. Header-Driven Development

The AI must never write `.cpp` implementation files until the C++23 `.hpp` API contract has been written, reviewed, and approved by the Lead Engineer.

