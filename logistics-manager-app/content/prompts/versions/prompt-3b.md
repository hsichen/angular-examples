# I want to add function/tool calling support for our fleet by the interactions SDK so that natural language chats with the model can become actions.

* Tools: 
    - Agent Skills:
        - `angular-developer`
        - `gemini-api-dev`
        - `gemini-interactions-api` 
    - MCP Servers:
        - Angular MCP
        - ChromeDevTools MCP

## Task

1. Review the Fleet related functions in the `server.ts` file to know what can be called and how to call them.

2. Outline a plan to implement this feature. 
    - Be sure to handle multi-turn tool calls and responses from the interactions API. Use the Gemini SDK skills for help on syntax and best practices where needed. Refer to the documention if you get stuck: https://ai.google.dev/gemini-api/docs/interactions.md.txt

    - Make a plan for a communication pattern with the UI so that the response from a tool call can create a "state" update on the UI. For example, if the user requests to add vehicles to the repair queue, the server will have an updated state, but the UI will be out of sync. Design a pattern using Angular resource to fetch data and a linkedSignal to manage the local UI state. When a tool call (e.g., add_to_repair_queue) succeeds, the linkedSignal must automatically reflect the updated fleet status from the server response.
    
    - Ensure the **`tools` array** uses the new format (an array of objects with `type: 'function'`).
    
    - **CRITICAL FIX**: For the **`input` on subsequent turns**, provide an array of objects where `type: 'function_result'` and the `result` field is always a **JSON-serializable object** (wrap arrays in `{ data: [...] }`). Do NOT wrap the array in a `role`/`parts` object.

3. For verification:
    - build the app and confirm that the build is error free. Use the Angular MCP server tools.
    - you may also use the ChromeDevTools MCP server to preview the build in a browser