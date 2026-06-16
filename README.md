# AI Agent Workflow Automation (n8n + Human-in-the-Loop)

An end-to-end automated social media posting pipeline built using n8n, Groq LLM, Cloudinary, and Gmail with a Human-in-the-Loop (HITL) approval system.

## 🚀 Demo Video
[Insert your Loom/Drive/YouTube Link Here]

## 🛠️ Architecture Explanation

### 1. Prompt Engineering Strategy (90-150 Word Limit)
To strictly enforce the word count constraint without abrupt text truncation, a multi-layered prompt engineering approach was used within the **Basic LLM Chain** powered by the **Groq Chat Model**. 
* **The Strategy:** The LLM is explicitly instructed with quantitative limits and strict formatting rules: *"Generate a highly engaging social media description on the given topic. The copy must strictly contain between 90 and 150 words. Ensure the content naturally concludes its logical paragraph within this boundary. Do NOT cut off mid-sentence or truncate arbitrarily."* 
* This dual approach (setting a floor and a ceiling while demanding structural closure) forces the model to self-regulate word output during token generation.

### 2. Programmatic Image Compositing Pipeline
To ensure strict brand compliance and eliminate the unpredictability of AI art generation, the image-creation stage is offloaded to a programmatic asset pipeline using **Cloudinary's Template and Transformation API**.
* **Logo Placement (Top-Right):** Inside the Cloudinary studio template (`t_myimage`), the static brand logo overlay is hard-mapped using a Compass grid set to **North-East (NE)** with subtle X/Y offsets for clear margins.
* **Typography Placement (Lower-Middle):** The text overlay is dynamically layered over the asset by injecting the topic name and binding it to the **South (S)** sector of the gravity configuration, supplemented with a baseline Y-offset (`g_south,y_50`). This approach guarantees identical structural symmetry across every programmatic execution regardless of textual length.

### 3. Pausing and Webhook Response Mechanism (HITL)
The Human-in-the-Loop framework leverages n8n's asynchronous execution state management using a decoupled pairing of **Gmail** and **Wait** nodes.
* **Mechanism:** When a post is generated, the **Gmail node** dispatches a structured HTML evaluation email. An approval button inside the email maps dynamically to n8n's internal execution context via `{{ $execution.resumeUrl }}`.
* **State Interruption:** Immediately after the email is sent, the workflow passes into the **Wait Node (On Webhook Call)**. This halts the n8n execution state in a pending cycle. When the reviewer clicks the link in their inbox, it triggers an inbound `GET` webhook response that securely tells the orchestrator to resume the workflow from its exact paused state, pushing the verified payload directly to the final **HTTP Request** mock publishing node.
