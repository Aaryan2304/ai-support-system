# Production-Ready Multi-Agent Customer Support System
## Strategic Implementation Roadmap

> **Timeline**: 24-36 hours | **Target**: Senior ML Engineer Evaluation  
> **Stack**: Zero-cost (Google Gemini, Neon PostgreSQL, Vercel)  
> **Focus**: Production patterns, robust error handling, observable architecture

---

## Executive Summary

Build an enterprise-grade multi-agent customer support system demonstrating production ML engineering patterns: LLM-based intent routing with confidence thresholds, tool orchestration with validation and audit trails, streaming responses with graceful degradation, and end-to-end type safety via Hono RPC monorepo (+30 guaranteed bonus points).

**Key Differentiators for Senior Evaluation:**
- Vendor-agnostic LLM client (swap providers easily)
- Tool audit logging for observability
- Streaming SSE with typed events
- AI reasoning transparency panel
- Comprehensive error boundaries and fallbacks
- Controller-Service-Repository pattern

---

## Architecture Overview

```mermaid
flowchart TB
    subgraph Frontend["Frontend (Vite + React)"]
        UI[Chat UI]
        ReasoningPanel[Agent Reasoning Panel]
        Stream[SSE Client]
    end
    
    subgraph Backend["Backend (Hono on Vercel Edge)"]
        Controllers[Controllers Layer]
        Middleware[Middleware: Error, Rate Limit, Auth]
        
        subgraph Services["Services Layer"]
            RouterSvc[Router Service]
            AgentSvc[Agent Services]
            ToolSvc[Tool Services]
            LLMClient[LLM Client Interface]
        end
        
        subgraph Agents["Multi-Agent System"]
            Router[Router Agent]
            Support[Support Agent]
            Order[Order Agent]
            Billing[Billing Agent]
        end
        
        subgraph Tools["Tool Layer"]
            ConvHistory[Query Conversation History]
            OrderTools[Get/Update Order Status]
            BillingTools[Invoice/Refund Tools]
        end
    end
    
    subgraph Data["Data Layer (Neon PostgreSQL)"]
        DB[(Database)]
        Seed[Seed Data: Orders, Invoices, Conversations]
    end
    
    subgraph External["External Services"]
        Gemini[Google Gemini API]
    end
    
    UI --> Stream
    Stream -->|Hono RPC| Controllers
    Controllers --> Middleware
    Middleware --> Services
    RouterSvc --> Router
    Router -->|delegate| Support
    Router -->|delegate| Order
    Router -->|delegate| Billing
    Support --> ConvHistory
    Order --> OrderTools
    Billing --> BillingTools
    Tools --> DB
    AgentSvc --> LLMClient
    Router --> LLMClient
    LLMClient --> Gemini
    ReasoningPanel -.->|displays| RouterSvc
```

---

## Core Design Principles

| Principle | Implementation |
|-----------|----------------|
| **Separation of Concerns** | Controllers (thin) → Services (business logic) → Tools (data access) |
| **Type Safety** | Hono RPC for client-server contracts, Zod schemas for LLM validation |
| **Graceful Degradation** | LLM fails → fallback routing → clarifying question → human handoff |
| **Observability** | Request IDs, tool audit logs, router decision tracking |
| **Cost Awareness** | Free-tier optimized (short prompts, response caching, quota monitoring) |
| **Production Ready** | Error boundaries, retry logic, rate limiting, idempotency |

---

## Technology Stack (Zero-Cost)

### Infrastructure
| Component | Technology | Justification |
|-----------|-----------|---------------|
| **LLM** | Google Gemini 1.5 Flash | 1500 req/day free, function calling support, fast streaming |
| **Database** | Neon PostgreSQL | 512 MB, unlimited pooled connections, no cold starts |
| **ORM** | Drizzle | Lighter than Prisma, better TypeScript inference for monorepo |
| **Backend** | Hono on Vercel Edge | Streaming SSE support, no cold starts, RPC integration |
| **Frontend** | Vite + React | Fast dev, simple deployment |
| **Monorepo** | Turborepo | Guaranteed +30 bonus, shared types |
| **Deployment** | Vercel (all-in-one) | Free tier, native streaming, simple CI/CD |

### Development Tools
- **Validation**: Zod (runtime schema validation)
- **Rate Limiting**: Token bucket algorithm (in-memory)
- **Testing**: Vitest (unit), Playwright (integration)
- **CI/CD**: GitHub Actions
- **Video**: Loom (free tier)

---

## Project Structure (Turborepo Monorepo)

```
ai-support-system/
├── apps/
│   ├── api/                          # Hono backend on Vercel Edge
│   │   ├── src/
│   │   │   ├── controllers/          # Thin HTTP handlers
│   │   │   │   ├── chat.controller.ts
│   │   │   │   ├── agent.controller.ts
│   │   │   │   └── health.controller.ts
│   │   │   ├── services/             # Business logic
│   │   │   │   ├── router.service.ts
│   │   │   │   ├── agents/
│   │   │   │   │   ├── base.agent.ts
│   │   │   │   │   ├── support.agent.ts
│   │   │   │   │   ├── order.agent.ts
│   │   │   │   │   └── billing.agent.ts
│   │   │   │   ├── tools/
│   │   │   │   │   ├── conversation.tools.ts
│   │   │   │   │   ├── order.tools.ts
│   │   │   │   │   └── billing.tools.ts
│   │   │   │   └── llm/
│   │   │   │       └── llm.client.ts  # Vendor-agnostic interface
│   │   │   ├── middleware/
│   │   │   │   ├── error.middleware.ts
│   │   │   │   ├── ratelimit.middleware.ts
│   │   │   │   └── auth.middleware.ts
│   │   │   ├── lib/
│   │   │   │   ├── validation.ts
│   │   │   │   └── logger.ts
│   │   │   └── index.ts              # Hono app + RPC export
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── web/                          # React frontend
│       ├── src/
│       │   ├── components/
│       │   │   ├── Chat.tsx
│       │   │   ├── MessageList.tsx
│       │   │   ├── MessageInput.tsx
│       │   │   ├── ConversationList.tsx
│       │   │   ├── AgentReasoningPanel.tsx  # Bonus: Show AI decisions
│       │   │   └── TypingIndicator.tsx
│       │   ├── hooks/
│       │   │   ├── useSSE.ts         # Server-Sent Events hook
│       │   │   └── useConversation.ts
│       │   ├── lib/
│       │   │   └── api.client.ts     # Hono RPC client
│       │   ├── App.tsx
│       │   └── main.tsx
│       ├── package.json
│       └── tsconfig.json
│
├── packages/
│   ├── shared/                       # Shared types (Hono RPC)
│   │   ├── src/
│   │   │   ├── types/
│   │   │   │   ├── agent.types.ts
│   │   │   │   ├── conversation.types.ts
│   │   │   │   └── tool.types.ts
│   │   │   └── schemas/
│   │   │       ├── message.schema.ts
│   │   │       └── router.schema.ts
│   │   ├── package.json
│   │   └── tsconfig.json
│   │
│   └── database/                     # Drizzle + seed data
│       ├── src/
│       │   ├── schema/
│       │   │   ├── user.ts
│       │   │   ├── conversation.ts
│       │   │   ├── message.ts
│       │   │   ├── order.ts
│       │   │   ├── invoice.ts
│       │   │   └── tool-log.ts       # Audit trail
│       │   ├── seed.ts               # Realistic mock data
│       │   ├── client.ts
│       │   └── index.ts
│       ├── drizzle.config.ts
│       └── package.json
│
├── turbo.json
├── package.json
└── README.md
```

---

## Data Model (Drizzle Schema)

### Core Tables

```typescript
// User & Authentication
users {
  id: uuid (PK)
  email: string (unique)
  name: string
  created_at: timestamp
}

// Conversations
conversations {
  id: uuid (PK)
  user_id: uuid (FK)
  title: string?
  summary: string?              // For context compaction
  token_count: integer         // Track token usage
  created_at: timestamp
  updated_at: timestamp
}

// Messages
messages {
  id: uuid (PK)
  conversation_id: uuid (FK)
  sender: enum('user', 'agent', 'system')
  content: text
  agent_type: enum('router', 'support', 'order', 'billing')?
  metadata: jsonb              // Store tool calls, reasoning
  created_at: timestamp
}

// Business Domain
orders {
  id: uuid (PK)
  order_number: string (unique)
  user_id: uuid (FK)
  status: enum('pending', 'processing', 'shipped', 'delivered', 'cancelled')
  items: jsonb
  total: decimal
  tracking_number: string?
  created_at: timestamp
  updated_at: timestamp
}

invoices {
  id: uuid (PK)
  invoice_number: string (unique)
  user_id: uuid (FK)
  order_id: uuid (FK)?
  amount: decimal
  status: enum('paid', 'pending', 'overdue', 'refunded')
  due_date: date
  paid_at: timestamp?
  created_at: timestamp
}

refunds {
  id: uuid (PK)
  invoice_id: uuid (FK)
  amount: decimal
  status: enum('pending', 'approved', 'rejected', 'completed')
  reason: text
  idempotency_key: string (unique)  // Prevent duplicate refunds
  requested_at: timestamp
  processed_at: timestamp?
}

// Observability & Audit
tool_logs {
  id: uuid (PK)
  conversation_id: uuid (FK)
  message_id: uuid (FK)?
  tool_name: string
  input_params: jsonb
  output_result: jsonb
  execution_time_ms: integer
  status: enum('success', 'error')
  error_message: text?
  created_at: timestamp
}
```

### Seed Data Requirements
- **50+ orders** across all statuses (15% pending, 60% delivered, 10% cancelled, etc.)
- **30+ invoices** with varied statuses (80% paid, 10% pending, 5% refunded, 5% overdue)
- **10+ conversations** with 5-20 messages each (multi-turn context)
- **5-10 users** with realistic profiles
- **20+ refund records** (mix of pending/completed)

---

## API Specification (11 Required Endpoints)

### Health & Meta
```
GET  /api/health
     Response: { status: 'ok', db: 'connected', llm: 'available', timestamp: ISO }
```

### Agent Information
```
GET  /api/agents
     Response: { agents: [{ type, name, description, capabilities }] }

GET  /api/agents/:type/capabilities
     Response: { type, tools: [{ name, description, parameters }], examples }
```

### Chat & Conversations
```
POST /api/chat/messages
     Body: { conversationId?, message: string }
     Response: SSE stream with events:
       - event: typing
       - event: routing        data: { agent, confidence, reasoning }
       - event: tool_call      data: { tool, params }
       - event: partial        data: { text }
       - event: final          data: { messageId, fullText }
       - event: error          data: { message }

GET  /api/chat/conversations
     Query: ?userId=string
     Response: { conversations: [{ id, title, lastMessage, updatedAt }] }

GET  /api/chat/conversations/:id
     Response: { conversation, messages: [...], tokenCount }

DELETE /api/chat/conversations/:id
       Response: { deleted: true }
```

---

## Multi-Agent System Design

### Router Agent (Intent Classifier)

**Responsibility**: Analyze incoming query, classify intent, delegate to specialist agent

**Implementation** (Simplified for 24-36h timeline):
```typescript
// Simplified LLM-based routing (skip kNN/embeddings initially)
const routerPrompt = `You are an intent classifier for customer support.
Analyze the user message and return ONLY valid JSON matching this schema:

{
  "targetAgent": "SUPPORT" | "ORDER" | "BILLING" | "UNCLASSIFIED",
  "confidence": 0.0-1.0,
  "reasoning": "brief explanation",
  "actionableData": {
    "orderNumber": string | null,
    "invoiceId": string | null
  }
}

Rules:
- ORDER: order status, tracking, modifications, cancellations
- BILLING: payments, refunds, invoices, subscriptions
- SUPPORT: FAQs, troubleshooting, general inquiries
- UNCLASSIFIED: ambiguous or out-of-scope (ask clarifying question)

Context: Last 6 messages from conversation
User Message: "${userMessage}"
`;

// Validation with retry logic
const response = await llmClient.generateStructured(routerPrompt, routerSchema);
if (response.confidence < 0.4) {
  return askClarifyingQuestion();
}
```

**Fallback Strategy**:
1. LLM returns invalid JSON → Retry once with explicit schema reminder
2. Second failure → Default to SUPPORT agent
3. Confidence < 0.4 → Ask clarifying question ("Are you asking about an order or billing?")
4. LLM service unavailable → Rule-based keyword matching (degraded mode)

---

### Support Agent

**Tools**:
- `queryConversationHistory(conversationId, limit?)` - Retrieve past messages
- `searchFAQ(query)` - Semantic search over FAQ database (future: use embeddings)

**System Prompt**:
```
You are a helpful customer support agent. You can access conversation history.
When users ask general questions, provide clear, friendly answers.
Use the conversation history tool to reference previous context.
If the issue requires order tracking or billing, explain that you'll connect them to a specialist.
```

**Error Handling**:
- Conversation history unavailable → Respond without context, mention temporary limitation
- FAQ search fails → Provide general guidance

---

### Order Agent

**Tools**:
- `getOrderDetails(orderNumber)` - Fetch order info from DB
- `checkDeliveryStatus(orderNumber)` - Get tracking info
- `updateOrderStatus(orderNumber, newStatus, reason)` - Cancel/modify (with validation)

**System Prompt**:
```
You are an order specialist. You can look up order details and tracking information.
Always verify the order number matches the expected format (ORD-XXXXX).
For cancellations, confirm if the order is in a cancellable state (pending/processing only).
Be empathetic about shipping delays.
```

**Tool Validation**:
```typescript
// Example: updateOrderStatus tool
{
  validate: (params) => {
    if (!params.orderNumber.match(/^ORD-\d{5}$/)) {
      throw new ToolValidationError('Invalid order number format');
    }
    // Check if status transition is allowed
    const order = await getOrder(params.orderNumber);
    if (order.status === 'delivered' && params.newStatus === 'cancelled') {
      throw new ToolValidationError('Cannot cancel delivered order');
    }
  },
  execute: async (params) => {
    // Log to tool_logs table
    // Update with idempotency check
  }
}
```

---

### Billing Agent

**Tools**:
- `getInvoiceDetails(invoiceId)` - Fetch invoice from DB
- `processRefund(invoiceId, amount, reason, idempotencyKey)` - Initiate refund
- `checkRefundStatus(refundId)` - Get refund processing status

**System Prompt**:
```
You are a billing specialist. You can access invoice details and process refunds.
For refund requests:
1. Verify the invoice exists and is paid
2. Confirm the refund amount is valid (≤ invoice amount)
3. For amounts > $100, inform user it requires manager approval
4. Always use idempotency to prevent duplicate refunds
Be professional when discussing payment issues.
```

**Security & Idempotency**:
```typescript
processRefund: {
  validate: async (params) => {
    const invoice = await getInvoice(params.invoiceId);
    if (invoice.status !== 'paid') {
      throw new ToolError('Invoice must be paid to issue refund');
    }
    if (params.amount > invoice.amount) {
      throw new ToolError('Refund amount exceeds invoice total');
    }
    // Check for duplicate idempotency key
    const existingRefund = await findRefundByIdempotencyKey(params.idempotencyKey);
    if (existingRefund) {
      return existingRefund; // Return existing, don't create duplicate
    }
  },
  execute: async (params) => {
    // Create refund with status = 'pending'
    // If amount > $100, set requires_approval = true
    // Log to audit trail
  }
}
```

---

## Context Management & Token Optimization

### Strategy (Simplified for Timeline)

**Phase 1: Sliding Window (Implement First)**
```typescript
// Keep last N messages in prompt
const CONTEXT_WINDOW = 12;
const recentMessages = conversation.messages.slice(-CONTEXT_WINDOW);

// Estimate tokens (rough: 1 token ≈ 4 chars)
const estimateTokens = (text: string) => Math.ceil(text.length / 4);
const totalTokens = recentMessages.reduce((sum, msg) => 
  sum + estimateTokens(msg.content), 0
);
```

**Phase 2: Compaction Trigger** (Bonus if time permits)
```typescript
// When conversation exceeds threshold
if (conversation.message_count > 200 || totalTokens > 7000) {
  // Summarize old messages (keep recent 12)
  const oldMessages = conversation.messages.slice(0, -12);
  const summary = await llmClient.summarize(oldMessages);
  
  // Update conversation.summary
  await updateConversation(conversationId, { summary });
  
  // Archive old messages (soft delete or separate table)
  await archiveMessages(oldMessages.map(m => m.id));
}
```

**Future Enhancement** (Note in README):
- Add pgvector extension for semantic retrieval
- Retrieve top-K relevant messages instead of just recent
- Use embeddings for intent classification (hybrid router)

---

## Streaming Implementation (SSE)

### Backend (Hono)
```typescript
// In chat.controller.ts
export const streamMessage = async (c: Context) => {
  const { conversationId, message } = await c.req.json();
  
  return streamSSE(c, async (stream) => {
    try {
      // 1. Emit typing indicator
      await stream.writeSSE({
        event: 'typing',
        data: JSON.stringify({ isTyping: true })
      });

      // 2. Route to appropriate agent
      const routing = await routerService.classify(message, conversationId);
      await stream.writeSSE({
        event: 'routing',
        data: JSON.stringify(routing)
      });

      // 3. Execute agent with tools
      const agent = getAgent(routing.targetAgent);
      
      for await (const chunk of agent.executeStreaming(message, conversationId)) {
        if (chunk.type === 'tool_call') {
          await stream.writeSSE({
            event: 'tool_call',
            data: JSON.stringify(chunk.data)
          });
        } else if (chunk.type === 'text') {
          await stream.writeSSE({
            event: 'partial',
            data: JSON.stringify({ text: chunk.text })
          });
        }
      }

      // 4. Save to database
      const savedMessage = await messageService.create({
        conversationId,
        content: fullResponse,
        metadata: { routing, toolCalls }
      });

      // 5. Emit final event
      await stream.writeSSE({
        event: 'final',
        data: JSON.stringify({ messageId: savedMessage.id })
      });

    } catch (error) {
      await stream.writeSSE({
        event: 'error',
        data: JSON.stringify({ message: error.message })
      });
    }
  });
};
```

### Frontend (React)
```typescript
// useSSE hook
const useStreamingMessage = (conversationId: string) => {
  const [messages, setMessages] = useState<Message[]>([]);
  const [isTyping, setIsTyping] = useState(false);
  const [routing, setRouting] = useState<RoutingDecision | null>(null);
  
  const sendMessage = async (text: string) => {
    const eventSource = new EventSource(
      `/api/chat/messages?conversationId=${conversationId}&message=${text}`
    );
    
    eventSource.addEventListener('typing', (e) => {
      setIsTyping(true);
    });
    
    eventSource.addEventListener('routing', (e) => {
      const data = JSON.parse(e.data);
      setRouting(data);
    });
    
    eventSource.addEventListener('partial', (e) => {
      const { text } = JSON.parse(e.data);
      // Update current message with partial text
      setMessages(prev => updateLastMessage(prev, text));
    });
    
    eventSource.addEventListener('final', (e) => {
      setIsTyping(false);
      eventSource.close();
    });
    
    eventSource.addEventListener('error', (e) => {
      setIsTyping(false);
      // Show error UI
      eventSource.close();
    });
  };
  
  return { messages, sendMessage, isTyping, routing };
};
```

---

## Error Handling & Middleware

### Global Error Middleware
```typescript
export const errorMiddleware = async (c: Context, next: Next) => {
  const requestId = crypto.randomUUID();
  c.set('requestId', requestId);
  
  try {
    await next();
  } catch (error) {
    logger.error({
      requestId,
      error: error.message,
      stack: error.stack,
      path: c.req.path
    });
    
    // Classify error type
    if (error instanceof ValidationError) {
      return c.json({ error: error.message }, 400);
    } else if (error instanceof LLMTimeoutError) {
      return c.json({ 
        error: 'Service temporarily unavailable',
        requestId 
      }, 503);
    } else if (error instanceof RateLimitError) {
      return c.json({ error: 'Rate limit exceeded' }, 429);
    } else {
      // Don't leak internal errors
      return c.json({ 
        error: 'Internal server error',
        requestId 
      }, 500);
    }
  }
};
```

### Rate Limiting (Bonus)
```typescript
// Token bucket implementation (in-memory for simplicity)
class RateLimiter {
  private buckets = new Map<string, TokenBucket>();
  
  async checkLimit(userId: string): Promise<boolean> {
    const bucket = this.buckets.get(userId) ?? new TokenBucket({
      capacity: 100,      // 100 requests
      refillRate: 10,     // 10 per minute
    });
    
    this.buckets.set(userId, bucket);
    return bucket.consume(1);
  }
}

export const rateLimitMiddleware = async (c: Context, next: Next) => {
  const userId = c.get('userId') ?? c.req.header('x-forwarded-for');
  
  if (!await rateLimiter.checkLimit(userId)) {
    throw new RateLimitError('Too many requests');
  }
  
  await next();
};
```

---

## Testing Strategy

### Unit Tests (Vitest)
```typescript
// tests/unit/router.service.test.ts
describe('RouterService', () => {
  it('should classify order query correctly', async () => {
    const result = await routerService.classify(
      "What's the status of order ORD-12345?"
    );
    expect(result.targetAgent).toBe('ORDER');
    expect(result.confidence).toBeGreaterThan(0.8);
    expect(result.actionableData.orderNumber).toBe('ORD-12345');
  });
  
  it('should fallback to SUPPORT on low confidence', async () => {
    const result = await routerService.classify("hello");
    expect(result.targetAgent).toBe('SUPPORT');
  });
  
  it('should retry on invalid JSON from LLM', async () => {
    // Mock LLM to return invalid JSON first time
    mockLLMClient
      .mockResolvedValueOnce('invalid json')
      .mockResolvedValueOnce(JSON.stringify(validRouting));
    
    const result = await routerService.classify("test query");
    expect(mockLLMClient).toHaveBeenCalledTimes(2);
  });
});

// tests/unit/tools/order.tools.test.ts
describe('Order Tools', () => {
  it('should fetch order details from DB', async () => {
    const order = await orderTools.getOrderDetails('ORD-12345');
    expect(order.orderNumber).toBe('ORD-12345');
  });
  
  it('should throw error for invalid order number', async () => {
    await expect(
      orderTools.getOrderDetails('INVALID')
    ).rejects.toThrow('Invalid order number format');
  });
  
  it('should prevent cancelling delivered order', async () => {
    await expect(
      orderTools.updateOrderStatus('ORD-99999', 'cancelled')
    ).rejects.toThrow('Cannot cancel delivered order');
  });
});
```

### Integration Tests (Playwright/Supertest)
```typescript
// tests/integration/chat-flow.test.ts
describe('Chat Flow Integration', () => {
  beforeAll(async () => {
    // Seed test database
    await seedTestData();
  });
  
  it('should complete order inquiry flow', async () => {
    // 1. Create conversation
    const conv = await request(app)
      .post('/api/chat/messages')
      .send({ message: 'Where is my order ORD-12345?' })
      .expect(200);
    
    // 2. Verify routing
    expect(conv.body.routing.targetAgent).toBe('ORDER');
    
    // 3. Verify tool was called
    const logs = await db.query.toolLogs.findMany({
      where: eq(toolLogs.conversationId, conv.body.conversationId)
    });
    expect(logs).toHaveLength(1);
    expect(logs[0].toolName).toBe('getOrderDetails');
  });
  
  it('should handle concurrent messages gracefully', async () => {
    const conversationId = 'test-conv-123';
    
    // Send 5 messages simultaneously
    const promises = Array(5).fill(null).map((_, i) =>
      request(app)
        .post('/api/chat/messages')
        .send({ conversationId, message: `Message ${i}` })
    );
    
    const results = await Promise.allSettled(promises);
    
    // All should succeed (no race conditions)
    expect(results.every(r => r.status === 'fulfilled')).toBe(true);
  });
});
```

### Test Coverage Goals
- **Router logic**: 90%+ (critical path)
- **Tool validation**: 100% (security-critical)
- **Error handling**: 80%+
- **Integration flows**: 5-7 critical scenarios

---

## Deployment Strategy (Zero-Cost)

### Infrastructure Setup

**1. Neon PostgreSQL**
```bash
# Sign up at https://neon.tech
# Create project: ai-support-system
# Copy connection string (pooled)
DATABASE_URL=postgresql://user:pass@ep-xxx.neon.tech/dbname?sslmode=require
```

**2. Google Gemini API**
```bash
# Get API key from https://aistudio.google.com/app/apikey
GEMINI_API_KEY=your_api_key_here
```

**3. Vercel Deployment**
```bash
# Install Vercel CLI
npm i -g vercel

# Deploy from monorepo root
vercel --prod

# Set environment variables in Vercel dashboard
# - DATABASE_URL
# - GEMINI_API_KEY
```

### Deployment Configuration

**vercel.json**
```json
{
  "version": 2,
  "builds": [
    {
      "src": "apps/api/src/index.ts",
      "use": "@vercel/node"
    },
    {
      "src": "apps/web/package.json",
      "use": "@vercel/static-build"
    }
  ],
  "routes": [
    {
      "src": "/api/(.*)",
      "dest": "apps/api/src/index.ts"
    },
    {
      "src": "/(.*)",
      "dest": "apps/web/$1"
    }
  ]
}
```

**Environment Variables**
```bash
# .env.example
DATABASE_URL=postgresql://...
GEMINI_API_KEY=...
NODE_ENV=production
RATE_LIMIT_ENABLED=true
LOG_LEVEL=info
```

---

## Bonus Features (Prioritized by ROI)

### ✅ High Priority (Implement if time permits)

**1. Rate Limiting (2h)** - Easy win, shows production thinking
```typescript
// Already outlined above in middleware section
// Token bucket algorithm, per-user limits
```

**2. AI Reasoning Panel (2h)** - Major differentiator for evaluation
```typescript
// Component to display:
// - Router decision (agent, confidence, reasoning)
// - Tool calls made (name, params, results)
// - Token usage
// - Response time
```

**3. Context Compaction (3-4h)** - Demonstrates ML depth
```typescript
// Sliding window + summarization when hitting limits
// Track token count per conversation
```

### ⚠️ Medium Priority (Note in README as "Future Work")

**4. Integration Tests (3h)**
```typescript
// 5-7 critical flow tests
// Database seeding for tests
// GitHub Actions CI
```

**5. useworkflow.dev Integration**
```typescript
// Workflow orchestration for complex multi-step tasks
// Could replace manual agent routing logic
// Time-intensive for first-time use
```

### ❌ Skip for Timeline

**6. pgvector + Hybrid Router**
- Excellent for production, adds 4-6h complexity
- Note in README as "Production Enhancement"

**7. Comprehensive Observability**
- Prometheus/Grafana setup
- Use simple console logging for demo

---

## Edge Cases & Mitigation

| Edge Case | Mitigation Strategy |
|-----------|-------------------|
| **LLM returns invalid JSON** | Validate with Zod → Retry once with schema reminder → Fallback to SUPPORT agent |
| **LLM service timeout/unavailable** | 10s timeout → Exponential backoff → Fallback to rule-based routing (keyword matching) |
| **Database connection fails** | Connection pooling (Neon) → Retry with backoff → Return graceful error "Service temporarily unavailable" |
| **Token limit exceeded (context too large)** | Track token count → Trigger summarization at 80% limit → Implement sliding window |
| **User sends rapid messages (spam)** | Rate limiting middleware → Token bucket (100 req/hr) → Return 429 with retry-after header |
| **Stream interruption mid-response** | Wrap in try/catch → Save partial response → Allow resume/retry |
| **Concurrent messages to same conversation** | Database transactions → Optimistic locking → Queue processing if needed |
| **Tool execution timeout** | 2s timeout per tool → Cancel gracefully → Inform user "Lookup taking longer than expected" |
| **Invalid tool parameters from LLM** | Zod validation before execution → Return error to LLM for self-correction → Max 2 retries |
| **Duplicate refund requests** | Idempotency keys → Check existing refunds before creating → Return existing result |
| **Order in invalid state for action** | State machine validation (e.g., can't cancel 'delivered') → Clear error message to user |
| **Router low confidence (<0.4)** | Ask clarifying question → Present option buttons → Default to SUPPORT with context |
| **Memory leak from abandoned streams** | Cleanup on disconnect → AbortController for each stream → Timeout inactive streams |

---

## Execution Timeline (24-36h)

### Phase 1: Foundation (8h)
**Hours 1-3**: Infrastructure
- [ ] Create Turborepo monorepo structure
- [ ] Set up Neon PostgreSQL + Drizzle schema
- [ ] Configure Hono app with basic middleware
- [ ] Set up Hono RPC between frontend and backend

**Hours 4-6**: Core Backend
- [ ] Implement LLMClient interface (Gemini integration)
- [ ] Build RouterService with LLM-based classification
- [ ] Create BaseAgent abstract class
- [ ] Implement tool validation layer

**Hours 7-8**: Database & Seeding
- [ ] Write seed script (50+ orders, 30+ invoices, 10+ conversations)
- [ ] Test database connections and queries

---

### Phase 2: Agent System (10h)
**Hours 9-12**: Agent Implementation
- [ ] Support Agent + conversation history tool
- [ ] Order Agent + order management tools (get, update, track)
- [ ] Billing Agent + billing tools (invoice, refund, status)
- [ ] Tool audit logging to tool_logs table

**Hours 13-16**: API Endpoints
- [ ] POST /api/chat/messages (SSE streaming)
- [ ] GET /api/chat/conversations/:id
- [ ] GET /api/chat/conversations
- [ ] DELETE /api/chat/conversations/:id
- [ ] GET /api/agents
- [ ] GET /api/agents/:type/capabilities
- [ ] GET /api/health

**Hours 17-18**: Error Handling
- [ ] Global error middleware
- [ ] Retry logic for LLM calls
- [ ] Fallback routing strategies

---

### Phase 3: Frontend & Integration (8h)
**Hours 19-22**: React UI
- [ ] Chat interface with message list
- [ ] Message input with typing indicator
- [ ] SSE client hook (useStreamingMessage)
- [ ] Conversation sidebar
- [ ] Basic styling (Tailwind CSS)

**Hours 23-24**: Integration
- [ ] Connect frontend to Hono RPC client
- [ ] Test all API endpoints
- [ ] Fix CORS/connection issues

**Hours 25-26**: Bonus Features
- [ ] Agent Reasoning Panel (show router decisions + tool calls)
- [ ] Rate limiting middleware
- [ ] Basic integration tests (3-5 scenarios)

---

### Phase 4: Polish & Delivery (6-10h)
**Hours 27-30**: Bug Fixes & Edge Cases
- [ ] Test edge cases from mitigation table
- [ ] Fix streaming issues
- [ ] Handle error states in UI
- [ ] Database transaction testing

**Hours 31-33**: Documentation
- [ ] Comprehensive README with:
  - Architecture diagram
  - Setup instructions (local + deployment)
  - Zero-cost stack explanation
  - API documentation
  - Testing guide
  - Future enhancements (pgvector, etc.)
- [ ] Code comments for evaluators
- [ ] Environment variable documentation

**Hours 34-36**: Deployment & Demo
- [ ] Deploy to Vercel (frontend + backend)
- [ ] Verify production environment
- [ ] Record 4-5 min Loom video:
  - Architecture walkthrough (1 min)
  - Happy path demo: order inquiry → agent routing → tool execution (2 min)
  - Edge case demo: LLM fallback, error handling (1 min)
  - Code highlights: router logic, tool validation (1 min)
- [ ] Final testing on production URL

---

## Evaluation Checklist (Grading Criteria)

### Core Requirements (Must-Have)
- [x] **Architecture**: Controller-Service pattern, clean separation
- [x] **Multi-Agent System**: Router + 3 sub-agents (Support, Order, Billing)
- [x] **Agent Tools**: All agents have DB-backed tools
- [x] **Database**: PostgreSQL with seeded mock data
- [x] **API Routes**: All 11 endpoints implemented
- [x] **Streaming**: SSE responses from AI agents
- [x] **Persistence**: Conversations and messages saved to DB
- [x] **Error Handling**: Middleware-based error handling throughout

### Bonus Points
- [x] **Monorepo + Hono RPC** (+30 points guaranteed)
- [x] **Rate Limiting** (Token bucket algorithm)
- [ ] **useworkflow.dev** (Skip for timeline, note as future work)
- [x] **Integration Tests** (5-7 critical scenarios)
- [x] **Context Management** (Sliding window + compaction trigger)
- [x] **AI Reasoning Display** (Transparency panel showing router decisions)
- [x] **Deployed Live Demo** (Vercel)

### Senior Engineer Expectations
- [x] **Vendor-Agnostic Design**: LLMClient interface (swap providers easily)
- [x] **Observability**: Tool audit logs, request IDs, router decision tracking
- [x] **Validation**: Zod schemas for all LLM outputs
- [x] **Idempotency**: Refund operations use idempotency keys
- [x] **Graceful Degradation**: Fallback strategies at every layer
- [x] **Type Safety**: End-to-end types via Hono RPC
- [x] **Edge Case Handling**: Comprehensive mitigation strategies
- [x] **Production Patterns**: Retry logic, circuit breakers, rate limiting

---

## README Structure (For Submission)

```markdown
# AI-Powered Multi-Agent Customer Support System

## Executive Summary
[1-2 paragraphs: what it does, why architecture choices matter]

## Architecture
[Mermaid diagram + explanation of agent routing, tool orchestration]

## Tech Stack (Zero-Cost)
[Table: Technology → Justification → Free Tier Limits]

## Getting Started
### Prerequisites
### Local Development
### Deployment (Vercel + Neon)

## API Documentation
[All 11 endpoints with examples]

## Multi-Agent System
### Router Agent Design
### Sub-Agent Capabilities
### Tool Implementation

## Key Features
- ✅ Hono RPC Monorepo (+30 Bonus Points)
- ✅ Streaming SSE Responses
- ✅ AI Reasoning Transparency
- ✅ Rate Limiting
- ✅ Context Management
- ✅ Comprehensive Error Handling

## Testing
[How to run tests, coverage report]

## Production Enhancements (Future Work)
- pgvector for semantic search
- Hybrid router (kNN + LLM)
- Comprehensive observability (Prometheus)
- Advanced context compression

## Demo Video
[Link to Loom walkthrough]

## Evaluation Notes for Reviewers
[Highlight architectural decisions, edge cases handled, bonus features]
```

---

## Final Notes for Implementation

### Critical Success Factors
1. **Prioritize Hono RPC monorepo first** - Guaranteed +30 points, foundational
2. **Test streaming early** - SSE can be tricky, don't leave for last minute
3. **Seed realistic data** - Evaluators will test edge cases (cancelled orders, overdue invoices)
4. **Deploy incrementally** - Push to Vercel at hour 16 to catch issues early
5. **Show your thinking** - AI reasoning panel differentiates you from other candidates

### Time Savers
- Use Drizzle Studio for database visualization during development
- Copy-paste prompt templates for agents (don't overthink initially)
- Use Vercel's environment variable UI (easier than CLI)
- Record Loom in 2-3 takes (don't aim for perfection)

### Red Flags to Avoid
- ❌ No error handling (instant fail for senior evaluation)
- ❌ Hardcoded routing (not using LLM for classification)
- ❌ Tools that never fail (unrealistic)
- ❌ No validation of LLM outputs (security risk)
- ❌ Missing audit trail (production systems need observability)

---

## Resources & Quick Links

**Documentation**
- [Hono](https://hono.dev/) - Backend framework
- [Hono RPC](https://hono.dev/guides/rpc) - Type-safe client
- [Drizzle ORM](https://orm.drizzle.team/) - Database toolkit
- [Vercel AI SDK](https://sdk.vercel.ai/docs) - LLM integration
- [Google Gemini API](https://ai.google.dev/docs) - Free LLM
- [Turborepo](https://turbo.build/repo/docs) - Monorepo tool

**Free Tier Limits**
- Gemini: 1,500 requests/day, 15 RPM
- Neon: 512 MB storage, unlimited connections (pooled)
- Vercel: 100 GB bandwidth, 6,000 build minutes/month

**Student Benefits**
- [GitHub Student Pack](https://education.github.com/pack) - Free Heroku credits, domains

**Templates & Examples**
- [Hono + Vercel Starter](https://github.com/honojs/starter)
- [Turborepo Hono Example](https://github.com/vercel/turbo/tree/main/examples)

---

## Success Metrics

**Must Achieve**:
- ✅ All 11 API endpoints functional
- ✅ Router correctly delegates to 3 agents
- ✅ Tools query actual database (not mocked)
- ✅ Streaming responses work end-to-end
- ✅ Deployed and accessible via public URL
- ✅ Loom video submitted (2-5 min)

**Exceeds Expectations**:
- ✅ Hono RPC monorepo implemented (+30 guaranteed)
- ✅ Rate limiting + AI reasoning panel (bonuses)
- ✅ Comprehensive error handling with fallbacks
- ✅ Integration tests demonstrating edge cases
- ✅ Professional README with architecture thinking

**Senior Engineer Recognition**:
- ✅ Vendor-agnostic abstractions (LLMClient interface)
- ✅ Observable system (audit logs, request tracking)
- ✅ Production-ready patterns (idempotency, validation, graceful degradation)
- ✅ Clear architectural documentation showing scale thinking

---

**This roadmap is designed for execution. Follow it sequentially, track progress, and ship incrementally. Good luck! 🚀**
