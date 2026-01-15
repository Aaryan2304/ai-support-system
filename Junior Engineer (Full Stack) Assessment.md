**Fullstack Engineering Assessment**
## **Overview** 
Build the fullstack project for an AI-powered customer support system with a multi-agent architecture. A router agent should analyze incoming queries and delegate to specialized sub-agents, each with access to relevant tools.The system should maintain conversational context across messages to enable more accurate and personalized responses.  

## **Requirements** 
### **Architecture** 
1. Controller-Service pattern 
1. Clean separation of concerns 
1. Proper error handling throughout (Recommended use a middleware) 
### **Multi-Agent System** 
**Router Agent (Parent)** 

- Analyzes incoming customer queries 
- Classifies intent and delegates to appropriate sub-agent 
- Handles fallback for unclassified queries 

**Sub-Agents (implement all three):** 

1. ***Support Agent*** 
- Handles general support inquiries, FAQs, troubleshooting 
- Tools: query conversation history 
2. ***Order Agent*** 
- Handles order status, tracking, modifications, cancellations 
- Tools: fetch order details, check delivery status 
3. ***Billing Agent*** 
- Handles payment issues, refunds, invoices, subscription queries 
- Tools: get invoice details, check refund status 
### **Agent Tools** 
4. Each sub-agent must have tools 
4. Tools should query actual data from database (mock data is enough) 
4. Seed database with sample orders, payments, conversations 
4. The agents should have a conversation context. On the User’s previous contexts. 
### **API & Database** 
8. RESTful API endpoints for chat interactions 
8. Streaming responses from AI agents (Recommended) 
8. Conversation and message persistence 
8. Real-time agent is typing indicator 

### **API Routes** 
/api 

├── /chat 

│   ├── POST /messages                    # Send new message 

│ ├── GET /conversations/:id              # Get conversation history

│ ├── GET /conversations                  # List user conversations

│ └── DELETE /conversations/:id           # Delete conversation

│

├── /agents

│ ├── GET /agents                         # List available agents

│ └── GET /agents/:type/capabilities      # Get agent capabilities

└── /health                               # Health check

### **Tech Stack** 

|**Frontend** |Basic UI (React / Vite) |
| - | - |
|**Backend** |[Hono.dev](https://hono.dev) |
|**Database** |PostgreSQL |
|**ORM** |Prisma / Drizzle |
|**AI** |Vercel AI SDK |

## **Bonus Points** 

|<p>**Hono RPC + Monorepo Setup (+30 points Guaranteed)**</p><p>Set up a monorepo with Hono RPC for end-to-end type safety between backend and frontend. Use <https://turborepo.dev/> for monorepo management. </p>|
| - |
|<p>**Other Bonuses**</p><p>1. Rate limiting implementation</p><p>2. Usage of <https://useworkflow.dev></p><p>3. Unit/integration tests</p><p>4. Context management / Compaction (When running out of tokens)</p><p>5. Show reasoning of the AI. Or a loader of random words like “Thinking”, “Searching”</p><p>6. Deployed live demo</p>|

## **Evaluation Focus** 
Backend architecture and code organization Multi-agent system design and routing logic Tool implementation and data flow 

API design and error handling 
## **Submission** 
12. GitHub repository with README
12. Loom video walkthrough (2-5 min)
12. Working setup instructions
