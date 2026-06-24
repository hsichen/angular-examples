# Add function/tool calling support for our fleet by the interactions SDK so that natural language chats with the model can become actions.

## Task

1. **Review** the fleet-related functions in `src/server.ts` to identify candidates for tool calling.

2. **Implement Multi-Turn Tool Support**:
    - Use the **Interactions API** stateful mode with `previous_interaction_id`.
    - **Flat Tool Definition**: Ensure the `tools` array contains flat objects (sibling `type`, `name`, `description`, and `parameters`).
    - **Mandatory Schema**: Every tool MUST have a `parameters` schema, even if it is an empty object: `parameters: { type: 'object', properties: {} }`.
    - **Function Results**: For subsequent turns, provide `input` as an array of objects where `type: 'function_result'`, using `call_id` (matching the model's output `id`), and a JSON-serializable `result`.

3. **UI State Synchronization**:
    - Design a pattern using Angular **`resource`** to fetch data.
    - Use **`linkedSignal`** in components to manage local UI state that tracks the resource values.
    - Automatically trigger resource `.reload()` in `FleetService` whenever a chat interaction resolves, ensuring tool-call side effects are reflected in the UI.

4. **Verification**:
    - Build the app and confirm it is error-free using Angular MCP tools.
    - Use ChromeDevTools MCP to verify end-to-end functionality (e.g., adding a ticket via chat and seeing it appear in the UI).

## Implementation Reference (Ground Truth)

```typescript
import { GoogleGenAI } from '@google/genai';
const client = new GoogleGenAI({});

// 1. Tool Definition (Flat structure + Mandatory parameters)
const tool = {
    type: 'function',
    name: 'get_status',
    description: 'Get current system status.',
    parameters: { type: 'object', properties: {} }
};

// 2. Initial Request
let interaction = await client.interactions.create({
    model: 'gemini-3-flash-preview',
    input: 'Status check.',
    tools: [tool]
});

// 3. Multi-turn Orchestration Loop
while (interaction.status === 'requires_action') {
    const outputs = interaction.outputs.filter(o => o.type === 'function_call');
    const results = outputs.map(call => ({
        type: 'function_result',
        name: call.name,
        call_id: call.id, // Critical mapping
        result: { data: 'All systems green' } // Must be JSON serializable
    }));

    interaction = await client.interactions.create({
        model: 'gemini-3-flash-preview',
        previous_interaction_id: interaction.id,
        input: results,
        tools: [tool] // Re-specify tools on every turn
    });
}
```
