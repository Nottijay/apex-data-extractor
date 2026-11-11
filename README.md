# Apex Logistics: Local Multi-Modal Data Extractor

## Overview
A fully local, automated ETL (Extract, Transform, Load) pipeline designed to process messy, multi-document images or text invoices. The system uses local vision (OCR) to read the documents and a local LLM to structure the data into strict JSON, complete with automated discrepancy checking.

## Architecture
- **Agent Framework:** Hermes Agent
- **Main Model:** Qwen3.6 35B-A3B (Mixture-of-Experts via Ollama)
- **Vision/OCR Engine:** PaddleOCR (Running as a local MCP Server)
- **Memory:** Default (Stateless)

## 🛠️ Environment & Tool Configuration (Crucial Guardrails)
To ensure the agent remained strictly focused on the ETL task and didn't hallucinate, the tool environment was heavily restricted.

**Tools ENABLED:**
- `File Operations`: Required to read text files and write the final JSON outputs.
- `Skills`: Required to load the `/read_image` and `/extract_invoice` instructions.
- `MCP / Tool Search`: Required to find and execute the local PaddleOCR tool.

**Tools DISABLED (and why):**
- `Vision / Image Analysis`: **Disabled.** If left on, the main LLM would bypass the OCR pipeline and use its own native vision, defeating the purpose of testing the OCR integration.
- `Web Search`: **Disabled.** Prevents the agent from trying to look up invoice math errors online instead of processing the data.
- `Code Execution / Terminal`: **Disabled.** Prevents the agent from trying to pip install random packages or write Python scripts when it gets stuck (a behavior observed in earlier testing).
- `Task Delegation`: **Disabled.** Keeps the pipeline 100% local and avoids API timeout loops.

## 🧠 Engineering Journey & Resolved Issues
This project was built through rigorous multi-layer debugging:

1. **The Cloud vs. Local Pivot:** Originally used Gemini API for vision, but suffered from severe latency and free-tier timeouts. Pivoted to a local PaddleOCR MCP server for 100% reliability and privacy.
2. **The Missing Engine Bug:** The OCR server was connecting but returning empty strings. Discovered the Python wrapper (`paddleocr`) was installed, but the underlying C++ engine (`paddlepaddle`) was missing.
3. **The Deferred Tool Architecture:** Hermes uses a "Deferred Tool" system to save context. The agent initially failed to find the OCR tool because it tried to call a hardcoded name. Fixed by updating the skill to use dynamic `tool_search` and `tool_call`.
4. **The Guardrail Escalation:** The agent hit an infinite loop trying to "fix" fake math on a test invoice. Resolved by moving error-handling constraints from the global `soul.md` into the specific `SKILL.md` file, and instructing the agent to flag discrepancies rather than correct them.

## 📊 Data Integrity (ETL Features)
Instead of blindly trusting the LLM, the pipeline enforces strict data integrity:
- Extracts data exactly as printed, even if the math is wrong.
- Calculates the `computed_total` based on line items.
- Compares it to the `printed_total`.
- Flags any mismatches in a `discrepancies` array.
- Sets `needs_human_review: true` if OCR confidence is low or math is broken.

## Repository Structure
- `soul.md`: The main system prompt (Manager/Router).
- `skills/read_image/SKILL.md`: The OCR pipeline instructions.
- `skills/extract_invoice/SKILL.md`: The JSON formatting and discrepancy flagging rules.
- `mock_invoice.txt`: Test data.
- `schema.json`: The target data structure.
- `output/`: The folder where the final JSON files are saved.