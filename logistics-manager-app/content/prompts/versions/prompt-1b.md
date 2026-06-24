# Add Gemini support to my Angular project's backend (Node.js).

Technical Stack:  
* Package: @google/genai (the new unified SDK)
* Model: gemini-3-flash-preview

## Task:

1. Review my current server entry point and suggest how to initialize the GoogleGenAI client.

2. Outline a plan for a new POST endpoint that accepts a prompt and returns a generated response using the interactions API method. Ensure the plan explains how the interaction_id will be stored or passed back to the Angular client to maintain session state.

3. Verify that endpoint is working by building the app, and use `curl` to ensure that the endpoint is working.