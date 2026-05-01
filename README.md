# Software Requirements Specification (SRS)

## 🟢 1. Introduction

### 1.1 Purpose
This Software Requirements Specification (SRS) defines the functional and non-functional requirements for the **AI-Based Study Assistant Chatbot** – an intelligent web application that provides real-time academic assistance through a conversational interface. The document is intended for:
* **Developers & Testers** – to guide implementation and verification.
* **Project Stakeholders** – to validate scope and acceptance criteria.
* **UX/UI Designers** – to align design decisions with required user interactions.

### 1.2 Project Scope
**The software will:**
* Allow students to create accounts, log in, and manage their sessions.
* Provide an interactive chat interface where a user types a query and receives an AI-generated academic answer.
* Integrate with an external NLP/Chatbot API (e.g., OpenAI API) to interpret queries and generate responses.
* Persist complete chat history per user, enabling review of past conversations.
* Present a simple, responsive web interface accessible via modern desktop and mobile browsers.
* Include basic administrative tools for user management and system health monitoring (recommended for complete deployment).

**The software will not:**
* Replace a full Learning Management System (LMS) or course delivery platform.
* Guarantee 100% factual accuracy of AI-generated answers (a disclaimer is required).
* Provide real-time human tutoring or video/audio communication.
* Support multimedia file uploads or generation (images, audio, video) in this version.
* Offer offline functionality – a live internet connection to the AI API is mandatory.

## 🟢 2. Overall Description

### 2.1 User Personas (Actors)

| Actor       | Description |
|-------------|-------------|
| **Student** | Primary end-user. Registers/logs in, submits study-related queries, views chat history, manages account. |
| **Admin**   | Manages user accounts (activate/deactivate), monitors system load, reviews flagged content, accesses usage statistics. |
| **AI Service** | External system (e.g., OpenAI API) that receives formatted prompts and returns text responses. Not a human actor, but an essential interfacing system. |

### 2.2 Operating Environment
* **Client-side:** The application shall run on the latest two versions of Google Chrome, Mozilla Firefox, Microsoft Edge, and Safari. JavaScript must be enabled. Responsive design supports viewports from 320px width (mobile) to 1920px+ (desktop).
* **Server-side:** The backend shall be deployed on a **Node.js** application server with access to a **MySQL** database. The server must have outbound HTTPS access to the chosen AI API.
* **Hosting:** The system is expected to be hosted on a cloud platform (e.g., AWS, Azure, Heroku) or a university server with proper SSL/TLS certificates.

### 2.3 Assumptions & Dependencies
**Assumptions:**
* Users possess basic digital literacy and can operate a web browser.
* Users have a stable internet connection.
* The AI API provider maintains 99.9% uptime and acceptable response latency.
* The institution will provide legal disclaimers about AI-generated advice.

**Dependencies:**
* Availability and active subscription of an external NLP/Chatbot API (e.g., OpenAI).
* External API’s rate limits, which will dictate concurrency ceilings.
* SMTP server (or equivalent) for sending account verification/notification emails.
* Browser support for WebSockets or HTTP long-polling if real-time token streaming is implemented (optional).

## 🟢 3. Functional Requirements (FRs)

Each requirement follows the format: **ID – Action – Actor – Expected Outcome.**

### 3.1 Module: User Authentication & Account Management

* **FR-01 (Student Registration):** The system shall allow a Visitor to create a new Student account by providing a unique email address, a display name, and a password satisfying minimum complexity rules (≥8 characters, 1 uppercase, 1 digit). Upon submission, the system shall send a verification email; the account remains inactive until verified.
* **FR-02 (Student Login):** The system shall allow a Registered Student to log in using email and password. After successful authentication, the system shall create a session and redirect the user to the chat interface.
* **FR-03 (Password Reset):** The system shall allow a Student to request a password reset link via registered email. The link must expire after 30 minutes and be usable only once.
* **FR-04 (Profile Management):** The system shall allow a logged-in Student to update their display name and password. Email change must require re-verification.
* **FR-05 (Admin User Management):** The system shall allow an Admin to list, search, deactivate, or permanently delete Student accounts. Deactivation shall prevent login and hide history until reactivation.

### 3.2 Module: Chat Interface

* **FR-06 (Send Message):** The system shall allow a logged-in Student to type a text query (up to 2,000 characters) and submit it via a “Send” button or Enter key. The system shall display the user’s message instantly in the chat view.
* **FR-07 (Receive AI Response):** Upon receiving a user query, the system shall transmit the query (with relevant conversation context, see FR-08) to the AI Service. The system shall display the returned AI response in the chat view, clearly distinguishing it from user messages. If the AI response is streamed token-by-token, the display must update incrementally.
* **FR-08 (Conversation Context):** The system shall include the last **N** exchanges (configurable, default 10) of the current conversation as context when calling the AI API, to enable context-aware answers. The system must not send context from other conversations.
* **FR-09 (New Conversation):** The system shall allow a Student to start a new conversation, clearing the current context. All historical conversations remain accessible under History.
* **FR-10 (Typing Indicator):** While waiting for the AI Service response, the system shall display an animated typing indicator in the chat area.
* **FR-11 (Error Handling – API Unavailable):** If the AI Service does not respond within 15 seconds, the system shall display a user-friendly error message: “The assistant is currently unavailable. Please try again later.” and automatically log the failure.
* **FR-12 (Error Handling – Inappropriate Content):** If the AI Service returns a response flagged for harmful content, the system shall replace the response with a generic message: “I’m unable to provide an answer to that.” and log the incident for Admin review.

### 3.3 Module: Chat History

* **FR-13 (Save Conversation):** The system shall automatically save each user query and its corresponding AI response (including timestamp) to the database under the authenticated Student’s account.
* **FR-14 (View History List):** The system shall provide a sidebar or dedicated page listing all past conversations, sorted by most recent activity. Each entry shall display the first query text and the date/time of the conversation start.
* **FR-15 (Load Past Conversation):** The system shall allow a Student to click a conversation from the history list and load the full chat transcript in the main chat view.
* **FR-16 (Search History):** The system shall allow a Student to search their chat history using keyword(s). Results shall show matching conversations with highlighted matches.
* **FR-17 (Delete Conversation):** The system shall allow a Student to delete an entire conversation permanently. This action requires confirmation.

### 3.4 Module: AI Integration

* **FR-18 (Prompt Sanitization):** Before sending a query to the AI Service, the system shall remove any personally identifiable information patterns (e.g., email addresses, phone numbers) from the user’s input to reduce accidental exposure.
* **FR-19 (System Prompting):** The system shall prepend a configurable system prompt (e.g., “You are a helpful academic assistant. Provide clear, factual explanations suitable for students.”) to every API call to constrain the AI’s behavior.
* **FR-20 (Fallback on Rate Limit):** If the AI Service returns a rate-limit error (HTTP 429), the system shall wait an exponential backoff period and retry up to 3 times before notifying the user of temporary unavailability.

### 3.5 Module: Administration (Recommended)

* **FR-21 (Admin Dashboard):** The system shall provide an Admin with a dashboard showing: total registered users, active conversations in the last 24 hours, AI API call count, and error rate.
* **FR-22 (Flagged Content Review):** The system shall allow an Admin to view a log of moderated responses (from FR-12) and optionally delete them or unflag them.

## 🟢 4. Non-Functional Requirements (NFRs)

| Category       | Requirement |
|----------------|-------------|
| **Performance** | **NFR-01:** Under normal load (≤100 concurrent active users), the system shall deliver an AI response (from submission to first token) in ≤3 seconds for 95% of requests, excluding network latency to the external API.<br>**NFR-02:** The web interface shall load within 2 seconds on a 4G mobile connection (time to interactive). |
| **Security**    | **NFR-03:** All communication between client and server shall be encrypted using TLS 1.2 or higher (HTTPS).<br>**NFR-04:** User passwords must be hashed using bcrypt (or equivalent) with a cost factor ≥10. API keys for the external AI service shall be stored in environment variables or a secrets vault, never in source code.<br>**NFR-05:** The system must not log raw user messages that could contain PII; logs must be stripped of message content or anonymized. |
| **Reliability** | **NFR-06:** The application server and database shall have a combined uptime of 99.5% (excluding planned maintenance and external API outages).<br>**NFR-07:** In the event of a database failure, the system shall recover within 1 hour from the latest backup, with no more than 15 minutes of lost chat data. |
| **Usability**   | **NFR-08:** The web interface must conform to WCAG 2.1 Level AA accessibility standards (e.g., sufficient color contrast, keyboard navigability, screen reader support for chat messages).<br>**NFR-09:** Error messages and tooltips must be written in plain English at a reading grade level ≤8 (e.g., Flesch-Kincaid). |
| **Scalability** | **NFR-10:** The system architecture must support horizontal scaling of the web server tier. The database must be index-optimized to handle chat history up to 10 million total messages without query times exceeding 500ms. |

## 🟢 5. Visual Model Recommendations

The following UML diagrams are recommended to accompany this SRS and clarify system structure and behavior.

1. **Use Case Diagram**  
   * **Actors:** Visitor, Student, Admin, AI Service (external system).  
   * **Use Cases:** Register, Login, Manage Profile, Send Message, View History, Search History, Start New Conversation, Manage Users (Admin), View Dashboard (Admin), Flagged Content Review (Admin).  
   * **Relations:** Include/extend relationships (e.g., “Send Message” includes “Receive AI Response”, “Login” extends “Verify Email”).

2. **Class Diagram (Conceptual Domain Model)**  
   * **Entities:** User (attributes: id, email, hashedPassword, displayName, isActive), Conversation (id, userId, title, createdAt), Message (id, conversationId, senderType [User/AI], content, timestamp, isFlagged).  
   * **Associations:** User 1–* Conversation, Conversation 1–* Message.  
   * Show multiplicities and key attributes; include Admin as specialization of User if needed.

3. **Entity-Relationship Diagram (ERD)** – for MySQL database  
   * **Tables:** Users, Conversations, Messages.  
   * **Relationships:** Users 1:N Conversations (FK user_id), Conversations 1:N Messages (FK conversation_id).  
   * **Indexes:** on user_id, conversation_id, timestamp, and full-text search index on message content.

4. **Sequence Diagram – Core Chat Flow**  
   * **Lifelines:** Student (Browser), Chatbot Frontend, Backend API, Database, AI Service.  
   * **Sequence:**  
     1. Student types query and clicks Send.  
     2. Frontend calls Backend API `POST /messages`.  
     3. Backend validates authentication, retrieves recent context from Database.  
     4. Backend sanitizes input, constructs prompt, calls AI Service (async).  
     5. AI Service returns response (possibly streamed).  
     6. Backend saves both query and response to Database.  
     7. Backend returns response to Frontend, which renders it incrementally.  
     8. Frontend updates chat view and scrolls to latest message.

5. **Activity Diagram – User Registration Process**  
   * **Flows:** Visitor fills form → validation → create account (pending) → send verification email → user clicks link → account activated → redirect to login.  
   * Include error paths: duplicate email, invalid token, expired link.

6. **State Diagram – Conversation State**  
   * **States:** Active (awaiting user input), Waiting for AI (query sent, awaiting response), Error (API failure/timeout), Completed (conversation ended or new one started).  
   * **Transitions:** User sends query (Active → Waiting), Response received (Waiting → Active), Timeout/error (Waiting → Error), User starts new chat (Active/Error → Completed).
