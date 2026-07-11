# n8n Workflows
 
This is my personal repository for open-source **n8n workflows** — automations I build and use myself, made available here so anyone can grab them, import them into their own n8n instance, and adapt them for their own use.
 
New workflows will be added here over time as I build them. Feel free to use, fork, or modify anything in this repo.
 
**Made by Jamil Ahmed**
🌐 Portfolio: [jamilahmed.dev](https://jamilahmed.dev) · [jamilahmed-portfolio.netlify.app](https://jamilahmed-portfolio.netlify.app/)
 
---
 
## Getting Started
 
### 1. Install n8n locally
 
If you don't have n8n set up yet, follow the [Install Guide](./install%20guide.md) in this repo — it covers both **Docker** (recommended) and **npm** installation, plus a production setup recommendation.
 
### 2. Import a workflow
 
1. Open your n8n instance (default: `http://localhost:5678`)
2. Go to **⋯ menu → Import from File**
3. Select the workflow `.json` file from the [`workflows`](./workflows) folder
4. Add any required credentials (API keys, tokens) — each workflow's own notes below will tell you what's needed
---
 
## Available Workflows
 
| Workflow | Description | Requirements |
|---|---|---|
| [WhatsApp LLM Chatbot](./workflows/WhatsApp_LLM_Chatbot_fixed.json) | Auto-replies to incoming WhatsApp messages using an AI model (Groq). Full setup guide: [whatsapp-chatbot-workflow-guide.md](./whatsapp-chatbot-workflow-guide.md) | Meta WhatsApp Cloud API access + Groq API key |
 
*(This table will be updated as new workflows are added.)*
 
---
 
## Contributing / Feedback
 
This is primarily a personal collection, but if you spot an issue with a workflow or have a suggestion, feel free to open an issue or pull request.
 
---
 
## License
 
Feel free to use and modify any workflow in this repo for personal or commercial use.
 
---
 
Built by **Jamil Ahmed** — [jamilahmed.dev](https://jamilahmed.dev) | [Portfolio](https://jamilahmed-portfolio.netlify.app/)
