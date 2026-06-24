# Optimized Gemini Interactions Tool-Calling Prompt

Use this prompt to implement or fix tool-calling logic using the unified Interactions API. It captures the specific requirements for state management, result formatting, and error prevention.

---

**Task:** Implement a multi-turn tool-calling loop using the `@google/genai` (v1.33+) Interactions API in Node.js.

**Technical Requirements:**
1. **Model:** Use `gemini-3-flash-preview`.
2. **State:** Maintain conversation state using `previous_interaction_id`.
3. **Tool Definition:** Use a **flat** `tools` array. Each tool must be an object with sibling `type: 'function'`, `name`, `description`, and a **mandatory** `parameters` schema (use `{ type: 'object', properties: {} }` for no-arg functions).
4. **Orchestration Loop:** 
   - Implement a manual `while (interaction.status === 'requires_action')` loop.
   - Re-specify the `tools` array on **every** turn inside the loop.
5. **Input Formatting (Subsequent Turns):** 
   - The `input` for tool-result turns must be an `Array` of objects.
   - Each object must have `type: 'function_result'` (use `as const` in TS).
   - The `call_id` must map exactly to the `id` provided by the model's `function_call` output.
6. **Result Wrapping (The "Data Wrapper" Pattern):**
   - The `result` field must **always** be a JSON-serializable **Object**.
   - **CRITICAL:** If a tool returns an Array, wrap it: `{ data: my_array }`.
   - **CRITICAL:** The Gemini API throws a 400 error if any field in the result is empty. If an array is empty, return a fallback: `{ data: 'No records found' }` or `{ success: true }`. 

**Goal:** Ensure the model can successfully chain multiple tool calls and provide a final natural language response to the user.

---

## Implementation Reference (Ground Truth)

```typescript
import { GoogleGenAI } from '@google/genai';
const ai = new GoogleGenAI({ apiKey: process.env['GEMINI_API_KEY'] });

const tools = [
  {
    type: 'function',
    name: 'get_data',
    description: 'Retrieve system data.',
    parameters: { type: 'object', properties: {} }
  }
] as const;

// 1. Initial interaction
let interaction = await ai.interactions.create({
  model: 'gemini-3-flash-preview',
  input: 'Run a status check.',
  tools: tools as any
});

// 2. Multi-turn Orchestration Loop
while (interaction.status === 'requires_action') {
  const functionCalls = (interaction.outputs || []).filter(o => o.type === 'function_call');
  
  const results = await Promise.all(functionCalls.map(async (call: any) => {
    // Execute your local tool logic here
    const rawResult = await myLocalToolHandler(call.name, call.arguments);
    
    // CRITICAL: Format result as a non-empty Object
    let result: any;
    if (Array.isArray(rawResult)) {
      result = { data: rawResult.length > 0 ? rawResult : 'No records found' };
    } else if (!rawResult) {
      result = { success: true };
    } else {
      result = typeof rawResult === 'object' ? rawResult : { value: rawResult };
    }

    return {
      type: 'function_result' as const,
      name: call.name,
      call_id: call.id, // Must match model's output id
      result
    };
  }));

  // 3. Send results back to the model
  interaction = await ai.interactions.create({
    model: 'gemini-3-flash-preview',
    previous_interaction_id: interaction.id,
    input: results,
    tools: tools as any // Re-specify tools
  });
}

// 4. Final output
console.log(interaction.outputs?.find(o => o.type === 'text')?.text);
```
