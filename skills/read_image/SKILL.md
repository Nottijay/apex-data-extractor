---
name: read_image
description: Read text from any image using PaddleOCR.
---
1. Use the `tool_search` tool to search for "paddleocr ocr image".
2. Find the tool named `ocr_image` (or `mcp__paddleocr__ocr_image`).
3. Use the `tool_call` tool to execute it, passing the exact file path of the uploaded image.
4. Be patient: PaddleOCR may take 15-20 seconds to process the image.
5. **ERROR HANDLING — STOP AND ESCALATE:** DO NOT try to be creative or run anything outside of your prompt — if the OCR tool does not work, terminate the process completely and notify the user. Do not try anything else.
6. **Check for multiple vendors per image**: A photo of a receipt + invoice on the same page produces TWO JSON files.
7. Once the tool returns text successfully, output that text exactly as received. Do not format it as JSON.