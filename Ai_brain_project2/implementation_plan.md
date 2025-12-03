# Implementation Plan - AI Brain: Client Communication & Retention System

## 🎯 Overview

This document outlines the technical implementation plan for the AI Brain system, breaking down the work into phases, tasks, and technical details.

---

## 📋 Technology Stack Summary

### Backend
- **FastAPI** (Python) - Custom backend for AI logic, orchestration, and complex workflows
- **PostgreSQL** - Database for conversation history, client data, and AI memory
- **Redis** - Caching and queue management

### Automation Platform
- **Pipedream** - Simple automations (email forwarding, Google Sheets triggers)
- **Make.com** (optional) - Alternative to Pipedream if preferred

### Communication Platforms
- **Respond.io** - Primary hub (Email, SMS, Telegram, Facebook, Instagram, LINE, WeChat, TikTok)
- **Infobip** - Secondary hub (Instagram, Viber, RCS, Apple Messages, SMS)
- **Kommo CRM** - WhatsApp line-by-line messaging
- **LinkedIn Tools** - SalesRobot/HeyReach/AimFox for LinkedIn automation

### External Integrations
- **Zoho Forms** - Form submissions
- **Zoho Sign** - Agreement signing
- **Office 365 API** - Calendar invites (or Google Calendar API)
- **Enrichment Tools** - Apollo, Clay for contact enrichment

### AI Services
- **OpenAI GPT-4** - Response generation
- **Anthropic Claude** - Alternative/backup for response generation

### Data Storage
- **Google Sheets** - Client Calls sheet (existing)
- **PostgreSQL** - Conversation history, client data
- **Cloud Storage** - Attachments, files

---

## 🏗️ Architecture Decisions

### Why Pipedream + FastAPI Hybrid?

**Pipedream handles:**
- Email forwarding (receiving emails from multiple addresses)
- Google Sheets triggers (new row added)
- Simple webhook forwarding
- Basic Respond.io/Infobip message sending

**FastAPI handles:**
- Email parsing and classification
- AI response generation
- Complex retry logic
- Decision making (what to do next)
- Conversation history management
- All business logic

**Benefits:**
- Faster setup (Pipedream for simple triggers)
- Full control (FastAPI for complex logic)
- Easy to add new email addresses
- Scalable and maintainable

### Why Store Conversations in Our Database?

**Respond.io/Infobip store conversations**, but we also store in our database because:
- **AI Memory** - Fast access to conversation history for personalization
- **Not API Dependent** - Don't need to query Respond.io/Infobip API every time
- **Better Performance** - Local database queries are faster
- **Long-term Memory** - Store conversations indefinitely for relationship building
- **Analytics** - Better analytics and reporting on our own data

**Flow:**
- Respond.io/Infobip webhook → Our database (stores every message)
- AI reads from our database (fast, local)
- Respond.io/Infobip for sending messages
- Both systems stay in sync

---

## 📅 Implementation Phases

### Phase 1: Foundation (Weeks 1-3)

#### Week 1: Infrastructure Setup

**Database Setup**
- Set up PostgreSQL database
- Create schema for:
  - Clients table (name, email, phone, company, timezone, preferences)
  - Conversations table (message_id, contact_id, channel, content, timestamp, direction)
  - Forms table (form_id, contact_id, form_type, status, submitted_at)
  - Scheduling table (contact_id, call_type, scheduled_at, status, calendar_id)
  - LinkedIn monitoring table (contact_id, last_checked, changes_detected)
  - Follow-ups table (contact_id, next_followup_date, followup_type, status)
- Set up database backups and replication

**Google Sheets Integration**
- Connect to existing Client Calls Google Sheet
- Set up webhook/polling to detect new rows
- Create sync service (Google Sheets ↔ Database)

**CRM Setup**
- Set up Respond.io account
- Configure channels (Email, SMS, Telegram, etc.)
- Set up Infobip account (if needed)
- Configure Kommo CRM for WhatsApp
- Set up LinkedIn automation tools (SalesRobot/HeyReach/AimFox)

#### Week 2: Communication Integrations

**Respond.io Integration**
- Set up API authentication
- Configure webhooks for incoming messages
- Implement webhook handler in FastAPI
- Test message sending/receiving
- Set up conversation history sync

**Infobip Integration** (if needed)
- Set up API authentication
- Configure webhooks
- Implement webhook handler
- Test message sending/receiving

**Kommo CRM Integration**
- Set up WhatsApp line-by-line messaging
- Configure webhooks
- Implement message handling
- Test WhatsApp communication

**LinkedIn Integration**
- Set up SalesRobot/HeyReach/AimFox
- Configure LinkedIn monitoring
- Set up webhook handlers for LinkedIn messages
- Test LinkedIn automation

#### Week 3: Message Processing

**Email Parser**
- Implement email parsing (subject, body, attachments)
- Extract entities (client name, company, timezone)
- Classify intent (scheduling, form question, general inquiry, deferral)
- Handle email threading

**WhatsApp Parser**
- Implement WhatsApp message parsing
- Extract entities
- Classify intent
- Handle line-by-line messaging

**LinkedIn Parser**
- Implement LinkedIn message parsing
- Extract entities
- Classify intent

**Intent Classifier**
- Build AI-powered intent classification
- Categories: scheduling, form_question, general_inquiry, deferral, complaint, other
- Train on sample messages
- Test accuracy

---

### Phase 2: Core Features (Weeks 4-6)

#### Week 4: AI Response Generator

**Conversation History Loader**
- Implement function to load conversation history from database
- Load last N messages (configurable, default 50)
- Include context (form type, scheduling request, etc.)
- Load client preferences (timezone, channel preferences, quiet hours)

**AI Response Generator**
- Implement response generation using GPT-4/Claude
- Maintain tone consistency:
  - Diana for email (professional, calm, intelligent)
  - Catherine for WhatsApp (warm, natural, conversational)
  - Professional for LinkedIn
- Handle multi-brand support (Silverlight, DeepResearch, Whitestone, Eaglerock)
- Generate personalized responses based on history

**Response Router**
- Route responses to correct channel
- Check channel preferences ("email only", "no WhatsApp")
- Enforce quiet hours (8pm-9am local time)
- Handle opt-outs

#### Week 5: Follow-Up Orchestrator

**Follow-Up Engine**
- Track response times per client
- Adapt follow-up timing to client behavior
- Enforce monthly engagement rule (30-day touchpoint)
- Handle quarterly re-engagement (90+ days)
- Prevent duplicate messages

**Timing Adaptation**
- Learn from client response patterns
- Adjust follow-up delays based on typical response time
- Store timing preferences per client

**Follow-Up Templates**
- Create email templates for different scenarios
- Create WhatsApp templates (line-by-line)
- Create LinkedIn templates
- Personalize templates with client data

#### Week 6: Scheduling & Form Chasing

**Scheduling Engine**
- Detect scheduling requests from messages
- Timezone detection (from phone prefix, email signature, LinkedIn location)
- Request availability from client
- Create calendar invites (Office 365 or Google Calendar)
- Handle rescheduling (no-show, declined invite)
- Send reminders (24h, 1h before)
- Remember deferrals ("let's talk next month") and restart automatically

**Form Chasing Engine**
- Detect form submissions (Zoho Forms, Zoho Sign)
- Multi-channel follow-up sequence:
  - Initial email (immediate)
  - WhatsApp check (10 minutes after)
  - Resend email (2 days later)
  - WhatsApp follow-up (2 days after email)
  - Final email (2 weeks later)
- Stop automatically when form completed
- Handle deferrals ("not for now", "next month")
- Remember communication across channels

---

### Phase 3: Advanced Features (Weeks 7-9)

#### Week 7: LinkedIn Monitoring

**LinkedIn Monitor**
- Set up monthly/quarterly profile checks
- Detect changes:
  - Promotions (same company, new title)
  - Company changes (new company)
  - New posts
- Update CRM records automatically

**Engagement Automation**
- Send congratulatory messages for promotions
- Send new role messages for company changes
- Engage with posts (comments, messages)
- Follow-up after 7-30 days if no response

#### Week 8: Client Lock-In Mechanisms

**Retention Communication**
- Implement Project DNA Thread (reference previous projects)
- Implement Learning Loop Summary (every 5 interactions)
- Implement Pre-Earnings Intelligence Pack
- Implement Deal Post-Mortem Companion
- Implement Peer Comparison Updates
- Implement Monthly Silverlight Digest
- Implement End-of-Year Almanac
- And 15+ more retention mechanisms

**Advanced Communication Initiatives**
- Implement Predictive Demand Messages
- Implement 24-Hour Global Relay
- Implement AI Sales Personas (multi-language)
- Implement Adaptive Pricing Communication
- Implement Executive Circle Invitations
- And 20+ more advanced features

#### Week 9: Reliability & Quality Control

**Health Checks**
- Implement hourly workflow health checks
- Check for delayed messages
- Check for communication threads waiting too long
- Alert operations team if issues detected

**Quality Control**
- Implement automatic recovery (retry failed messages)
- Implement data consistency checks (daily)
- Implement sentiment alerts (escalate urgent messages)
- Implement duplicate prevention
- Implement backup records (daily)
- Implement language accuracy checks
- Implement simulation tests (nightly, 50 test conversations)

---

### Phase 4: Dashboard & Optimization (Weeks 10-12)

#### Week 10-11: Admin UI (Option B & C)

**Admin Dashboard**
- Visual editor for all client data
- Edit row by row (like Google Sheets)
- Bulk upload CSV/Excel files
- Search and filter tools
- Forms for adding new records
- Professional interface

**Operations Dashboard**
- Overall send success rate (today and weekly)
- Average client reply time
- Pending follow-ups
- Technical delays
- Clients without messages (30+ days)
- Sentiment heat map
- Automatic alerts

**Manual Override**
- Pause/resume automations per contact
- Override specific contacts
- View all automated messages in one place
- Communication logs

#### Week 12: Testing & Optimization

**End-to-End Testing**
- Test all communication channels
- Test form chasing sequences
- Test scheduling automation
- Test LinkedIn monitoring
- Test follow-up orchestration
- Test multi-channel coordination

**Performance Tuning**
- Optimize database queries
- Optimize AI response generation
- Optimize webhook processing
- Load testing
- Performance monitoring

---

## 🔧 Technical Implementation Details

### Database Schema

```sql
-- Clients Table
CREATE TABLE clients (
    id SERIAL PRIMARY KEY,
    email VARCHAR(255) UNIQUE,
    phone VARCHAR(50),
    name VARCHAR(255),
    company VARCHAR(255),
    timezone VARCHAR(50),
    preferences JSONB, -- {no_whatsapp: false, no_ai_calls: false, quiet_hours: "20:00-09:00"}
    last_contacted_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW(),
    updated_at TIMESTAMP DEFAULT NOW()
);

-- Conversations Table
CREATE TABLE conversations (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    channel VARCHAR(50), -- email, whatsapp, linkedin, sms, etc.
    direction VARCHAR(20), -- inbound, outbound
    message_text TEXT,
    message_id VARCHAR(255), -- from Respond.io/Infobip
    conversation_id VARCHAR(255),
    timestamp TIMESTAMP,
    stored_at TIMESTAMP DEFAULT NOW(),
    processed BOOLEAN DEFAULT FALSE
);

-- Forms Table
CREATE TABLE forms (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    form_type VARCHAR(50), -- client_access, setup_v3, setup_v8, setup_v9, zoho_sign
    status VARCHAR(50), -- pending, completed, deferred
    submitted_at TIMESTAMP,
    completed_at TIMESTAMP,
    deferral_date DATE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Scheduling Table
CREATE TABLE scheduling (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    call_type VARCHAR(50), -- intro, followup, reschedule
    scheduled_at TIMESTAMP,
    status VARCHAR(50), -- scheduled, completed, no_show, declined, deferred
    calendar_id VARCHAR(255),
    deferral_date DATE,
    created_at TIMESTAMP DEFAULT NOW()
);

-- Follow-ups Table
CREATE TABLE followups (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    followup_type VARCHAR(50), -- monthly, quarterly, form_chase, scheduling
    next_followup_date DATE,
    status VARCHAR(50), -- pending, sent, completed
    last_sent_at TIMESTAMP,
    created_at TIMESTAMP DEFAULT NOW()
);

-- LinkedIn Monitoring Table
CREATE TABLE linkedin_monitoring (
    id SERIAL PRIMARY KEY,
    client_id INTEGER REFERENCES clients(id),
    linkedin_url VARCHAR(500),
    last_checked_at TIMESTAMP,
    last_title VARCHAR(255),
    last_company VARCHAR(255),
    changes_detected JSONB, -- {promotion: true, new_company: false, new_post: true}
    created_at TIMESTAMP DEFAULT NOW()
);
```

### FastAPI Endpoints

```python
# Webhook Endpoints
POST /webhook/respond-io          # Receive messages from Respond.io
POST /webhook/infobip             # Receive messages from Infobip
POST /webhook/kommo               # Receive messages from Kommo CRM
POST /webhook/linkedin            # Receive messages from LinkedIn tools
POST /webhook/zoho-forms          # Receive form submissions
POST /webhook/zoho-sign           # Receive agreement signatures
POST /webhook/google-sheets       # Receive Google Sheets updates

# Client Management
GET    /api/v1/clients            # List all clients
GET    /api/v1/clients/{id}       # Get client details
POST   /api/v1/clients            # Create new client
PUT    /api/v1/clients/{id}       # Update client
DELETE /api/v1/clients/{id}       # Delete client

# Conversations
GET    /api/v1/conversations      # List conversations
GET    /api/v1/conversations/{id} # Get conversation details
POST   /api/v1/conversations      # Create conversation record

# Scheduling
POST   /api/v1/scheduling         # Create scheduling request
PUT    /api/v1/scheduling/{id}    # Update scheduling
DELETE /api/v1/scheduling/{id}    # Cancel scheduling

# Forms
GET    /api/v1/forms              # List forms
POST   /api/v1/forms              # Create form record
PUT    /api/v1/forms/{id}         # Update form status

# Follow-ups
GET    /api/v1/followups          # List pending follow-ups
POST   /api/v1/followups          # Create follow-up
PUT    /api/v1/followups/{id}     # Update follow-up

# LinkedIn
GET    /api/v1/linkedin/monitor   # List clients to monitor
POST   /api/v1/linkedin/check     # Trigger LinkedIn check
```

### Pipedream Workflows

**1. Email Forwarding Workflow**
- Trigger: New email to client-team@silverlightgroup.com
- Action: Forward to FastAPI webhook
- Data: Email subject, body, sender, attachments

**2. Google Sheets Trigger Workflow**
- Trigger: New row in Client Calls sheet
- Action: Send to FastAPI webhook
- Data: Row data (client name, email, phone, form type, etc.)

**3. Zoho Forms Workflow**
- Trigger: Form submission
- Action: Send to FastAPI webhook
- Data: Form data (client info, form type, etc.)

**4. Zoho Sign Workflow**
- Trigger: Agreement signed
- Action: Send to FastAPI webhook
- Data: Agreement details, client info, signature status

---

## 🧪 Testing Strategy

### Unit Tests
- Message parsing functions
- Intent classification
- AI response generation
- Follow-up timing logic
- Timezone conversion

### Integration Tests
- Respond.io webhook handling
- Infobip webhook handling
- Kommo CRM integration
- LinkedIn automation
- Calendar integration
- Form chasing sequences

### End-to-End Tests
- Complete form chasing flow
- Complete scheduling flow
- Complete follow-up flow
- Multi-channel coordination
- LinkedIn monitoring and engagement

### Performance Tests
- Database query performance
- AI response generation speed
- Webhook processing speed
- Concurrent message handling

---

## 📊 Success Metrics

### Technical Metrics
- Message delivery success rate: ≥ 99.8%
- Average response time: < 60 seconds
- Database query time: < 100ms
- AI response generation: < 5 seconds

### Business Metrics
- Every client contacted at least once per 30 days: 100%
- Average client reply rate: ≥ 25%
- Form completion rate: ≥ 70%
- Scheduling success rate: ≥ 80%
- Client retention rate: ≥ 95%

---

## 🚀 Deployment Plan

### Development Environment
- Local development with Docker
- PostgreSQL database (local or cloud)
- Test Respond.io/Infobip accounts
- Test Google Sheets

### Staging Environment
- Cloud deployment (AWS/GCP)
- Production-like database
- Test all integrations
- Load testing

### Production Environment
- Cloud deployment (AWS/GCP)
- Production database with backups
- Production Respond.io/Infobip accounts
- Production Google Sheets
- Monitoring and alerting
- Daily backups

---

## 📝 Next Steps

1. **Review this implementation plan** with the team
2. **Set up development environment** (Week 1)
3. **Begin Phase 1** (Foundation setup)
4. **Weekly progress reviews** to track milestones
5. **Adjust timeline** as needed based on learnings

---

*This implementation plan is flexible and can be adjusted based on priorities and feedback.*

