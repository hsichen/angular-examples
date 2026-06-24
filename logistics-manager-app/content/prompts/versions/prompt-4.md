# I need to fix the Interactions API tool calling loop in `src/server.ts`. I am getting a `"Missing key 'type' in input 'content'"` error when passing the results of local tools back to the model.

## Please review the `gemini-interactions-api` and `gemini-api-dev` skills to understand how to correctly format the `input` on subsequent turns when using the `@google/genai` (v1.33+) Interactions API in Node.js. 

**Key Requirements:**
1. Use **`ai.interactions.create`** with the **`gemini-3-flash-preview`** model.
2. Pass the conversation state via **`previous_interaction_id`**.
3. Implement a **manual `while` loop** (since Node.js doesn't support automatic calling for local tools).
4. Ensure the **`tools` array** uses the new format (an array of objects with `type: 'function'`).
5. **CRITICAL FIX**: For the **`input` on subsequent turns**, provide an array of objects where `type: 'function_result'` and the `result` field is always a **JSON-serializable object** (wrap arrays in `{ data: [...] }`). Do NOT wrap the array in a `role`/`parts` object.
6. Map the model's **`id`** to the response's **`call_id`** exactly.

## Show me your plan before you make any changes so we can be on the same page