# Add communication from the client side app to the AI chat endpoint via the chat window in the web app. 

Task:
1. Review the endpoint and chat window code to understand where to integrate.

2. Outline a plan to implement the communication between the client and the server.
    - Use the **`@google/genai` interactions API**. Extract the generated message from the first text object in `interaction.outputs`. Refer to the documention if you get stuck: https://ai.google.dev/gemini-api/docs/interactions.md.txt

3. For verification:
    - build the app and confirm that the build is error free. Use the Angular MCP server tools.
    - you may also use the ChromeDevTools MCP server to preview the build in a browser
