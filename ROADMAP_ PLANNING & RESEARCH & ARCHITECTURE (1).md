# **App Architecture**
- [ ] Build a mental flow map
- [x] Design database
- [ ] Define main API endpoints
- [ ] Choose hosting and ORM
***
## Build a mental flow map\: 
```text
screen ➡ action️️️️ ➡ result
```
***
## Design database \(PostgreSQL via Prisma\)
```text
Tables: users, therapists, sessions, messages, feedbacks, routing_logs
```
### Step 1\. Domain entities and relationships
1. **<u>Entities</u>**
    * **User** — a registered client
    * **Therapist** — a therapist from the database
    * **Chat** — a communication channel between a specific User and Therapist
    * **Message** — a message within a Chat
    * **Feedback** — rating\/review of a session or therapist
2. **<u>Relationships</u>**
    * Each **User**\, after registration\, interacts with an **AI** agent → receives a **Therapist** matching their requirements\.
    * After the therapist is selected\, a new **Chat** is created \(User ↔ Therapist\)\.
    * Within each **Chat**\, the User and Therapist exchange **Messages**\.
    * After the session ends\, the User can leave **Feedback** tied to that specific **Chat**\.
### Step 2\. Database schema \(ERD\)
```text
User 1──◄ AI Routing ►──1 Therapist
User 1───∞ Chat ───1 Therapist
Chat 1──∞ Message
Chat 1──1 Feedback
```
* **AI Routing** is a logical step between **User** and **Therapist**\, not a separate entity in the database\.
* **Chat** saves the \"session\" of communication\, connects the pair **User**–**Therapist**\.
* **Message** are tied to **Chat** via `chat_id`\.
* **Feedback** also refers to `chat_id` to communicate with a specific session\.\*\*\*\*
**<u>Mermaid ER diagram \(Entity Relationship Diagram\)</u>**
```text
erDiagram
    
    %% Relationships between entities
    USER ||--o{ CHAT             : has         %% One user can have many chats
    THERAPIST ||--o{ CHAT        : serves      %% One therapist can serve many chats
    CHAT ||--o{ MESSAGE          : contains    %% One chat can contain many messages
    CHAT ||--o| FEEDBACK         : gathers     %% One chat may have one feedback
    USER ||--o{ FEEDBACK         : gives       %% One user can give many feedback entries
    USER ||--|| AUTH             : uses        %% One user has one authentication record

    %% User entity definition
    USER {
      UUID id PK                    %% Primary key: unique user ID
      String email                  %% User's email address
      String name                   %% Display name or full name
      String locale                 %% User's preferred language or region
      DateTime createdAt            %% Timestamp of creation
      DateTime updatedAt            %% Timestamp of last update
    }

    %% Authentication table separated for privacy/security
    AUTH {
      UUID userId PK FK             %% Foreign key to USER (also primary key for one-to-one)
      String passwordHash           %% Hashed password
      DateTime lastLogin?           %% Optional: track last login time
      Boolean isActive              %% Whether account is currently active
    }

    %% Therapist entity definition
    THERAPIST {
      UUID id PK                    %% Primary key: unique therapist ID
      String name                   %% Therapist's name
      String specialty              %% Area of expertise (e.g., anxiety, PTSD)
      Int    experienceYears        %% Years of professional experience
      String avatarUrl              %% Optional: URL to therapist's profile picture
      String locale                 %% Language/region for localization & matching
      DateTime createdAt            %% Timestamp of creation
      DateTime updatedAt            %% Timestamp of last update
    }

    %% Chat session between user and therapist
    CHAT {
      UUID id PK                    %% Primary key: chat ID
      UUID userId FK                %% Foreign key to USER
      UUID therapistId FK           %% Foreign key to THERAPIST
      Enum   status                 %% Chat status: "active", "archived", or "completed"
      DateTime createdAt            %% When the chat was initiated
      DateTime updatedAt            %% When the chat was last updated
    }

    %% Messages exchanged during chat
    MESSAGE {
      UUID id PK                    %% Primary key: message ID
      UUID chatId FK                %% Foreign key to CHAT
      Enum   senderType             %% "user" or "therapist" to identify the sender
      Text   content                %% Message body
      DateTime timestamp            %% When the message was sent
    }

    %% Feedback left by user at the end of a chat
    FEEDBACK {
      UUID id PK                    %% Primary key: feedback ID
      UUID chatId FK                %% Foreign key to CHAT (one-to-one relationship)
      UUID userId FK                %% Foreign key to USER
      Int    rating                 %% Rating score: 1 to 5
      Text   comment?               %% Optional comment field
      DateTime createdAt            %% When the feedback was submitted
    }
```
**<u>SQL script for creating your database tables in PostgreSQL\, with detailed inline comments explaining each table\, field\, and relationship</u>**
```SQL

-- Users table: stores application users
CREATE TABLE users (
    id UUID PRIMARY KEY,                        -- Unique user identifier
    email VARCHAR(255) NOT NULL UNIQUE,        -- User's email address, unique for login
    name VARCHAR(255) NOT NULL,                 -- User's display or full name
    locale VARCHAR(10) NOT NULL,                -- Preferred locale/language (e.g., 'en', 'uk')
    created_at TIMESTAMP NOT NULL DEFAULT NOW(), -- Timestamp when user was created
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()  -- Timestamp of last update
);

-- Authentication table: separated for security reasons (1:1 relationship with users)
CREATE TABLE auth (
    user_id UUID PRIMARY KEY REFERENCES users(id) ON DELETE CASCADE,  -- Links to user; cascade deletes
    password_hash TEXT NOT NULL,                                      -- Hashed password string
    last_login TIMESTAMP NULL,                                        -- Timestamp of last login (nullable)
    is_active BOOLEAN NOT NULL DEFAULT TRUE                           -- Whether user account is active
);

-- Therapists table: stores therapists' profiles
CREATE TABLE therapists (
    id UUID PRIMARY KEY,                                               -- Unique therapist identifier
    name VARCHAR(255) NOT NULL,                                        -- Therapist's full name
    specialty VARCHAR(255) NOT NULL,                                   -- Therapist's area of expertise
    experience_years INT NOT NULL,                                     -- Years of experience
    avatar_url TEXT,                                                   -- Optional link to therapist's avatar image
    locale VARCHAR(10) NOT NULL,                                       -- Therapist's language/locale
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),                       -- Timestamp of creation
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()                        -- Timestamp of last update
);

-- Enum type for chat status
CREATE TYPE chat_status AS ENUM ('active', 'archived', 'completed');

-- Chats table: sessions between user and therapist
CREATE TABLE chats (
    id UUID PRIMARY KEY,                                               -- Unique chat session ID
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,      -- User participating in chat
    therapist_id UUID NULL REFERENCES therapists(id) ON DELETE SET NULL, -- Therapist assigned; nullable if therapist is removed
    status chat_status NOT NULL DEFAULT 'active',                      -- Chat lifecycle status
    created_at TIMESTAMP NOT NULL DEFAULT NOW(),                       -- When chat was created
    updated_at TIMESTAMP NOT NULL DEFAULT NOW()                        -- Last update timestamp
);

-- Enum type for sender of messages
CREATE TYPE sender_type AS ENUM ('user', 'therapist');

-- Messages table: chat messages between user and therapist
CREATE TABLE messages (
    id UUID PRIMARY KEY,                                               -- Unique message ID
    chat_id UUID NOT NULL REFERENCES chats(id) ON DELETE CASCADE,      -- Chat to which this message belongs
    sender_type sender_type NOT NULL,                                 -- Who sent the message
    content TEXT NOT NULL,                                             -- Text content of the message
    timestamp TIMESTAMP NOT NULL DEFAULT NOW()                         -- When message was sent
);

-- Feedback table: stores user feedback per chat session
CREATE TABLE feedbacks (
    id UUID PRIMARY KEY,                                               -- Unique feedback ID
    chat_id UUID NOT NULL UNIQUE REFERENCES chats(id) ON DELETE CASCADE, -- Chat this feedback relates to
    user_id UUID NOT NULL REFERENCES users(id) ON DELETE CASCADE,     -- User who left the feedback
    rating INT NOT NULL CHECK (rating BETWEEN 1 AND 5),               -- Rating score (1 to 5)
    comment TEXT,                                                     -- Optional text comment
    created_at TIMESTAMP NOT NULL DEFAULT NOW()                       -- When feedback was created
);

-- Indexes can be added as needed for optimization
-- For example:
-- CREATE INDEX idx_chats_user_id ON chats(user_id);
-- CREATE INDEX idx_chats_therapist_id ON chats(therapist_id);
-- CREATE INDEX idx_messages_chat_id ON messages(chat_id);
-- CREATE INDEX idx_feedbacks_chat_id ON feedbacks(chat_id);

-- Triggers or application logic should keep 'updated_at' fields current on UPDATE
```
***
## Define main API endpoints
```text
/auth
/users
/therapists
/sessions
/messages
/feedback
/ai/symptoms
```
### User Endpoints
* `POST /users/register` 
    * Register new user \(email\, password\, name\, locale\)
* `POST /users/login` 
    * Authenticate user\, return auth token
* `GET /users/{id}`
    * Get user profile info \(id\, email\, name\, locale\)
* `PUT /users/{id}`
    * Update user profile \(name\, locale\)
* `DELETE /users/{id}`
    * Delete user account
### Authentication Endpoints
* Handled implicitly via `/users/login` and token management \(e\.g\.\, JWT refresh\, logout endpoints\)
### Therapist Endpoints
* `GET /therapists`
    * List all therapists \(optionally filtered by specialty\, locale\)
* `GET /therapists/{id}`
    * Get therapist profile
* *Admin only\:* `POST /therapists`
    * Create therapist profile
* `PUT /therapists/{id}`
    * Update therapist profile
* `DELETE /therapists/{id}`
    * Delete therapist profile
### Chat Endpoints
* `POST /chats`
    * Start a new chat session \(userId\, therapistId\)
* `GET /chats/{id}`
    * Get chat details\, including status
* `GET /users/{userId}/chats`
    * List all chats of a user
* `GET /therapists/{therapistId}/chats`
    * List all chats assigned to a therapist
* `PUT /chats/{id}/status`
    * Update chat status \(active\, archived\, completed\)
* `DELETE /chats/{id}`
    * Delete chat session
### Message Endpoints
* `POST /chats/{chatId}/messages`
    * Send a message in a chat \(senderType\, content\)
* `GET /chats/{chatId}/messages`
    * List all messages in a chat \(with pagination\)
### Feedback Endpoints
* `POST /chats/{chatId}/feedback`
    * Leave feedback \(userId\, rating\, comment\)
* `GET /chats/{chatId}/feedback`
    * Get feedback for a chat
* `GET /users/{userId}/feedback`
    * List all feedback given by a user`/{id}/status`
***
## Choose hosting and ORM
```text
Supabase as cloud-hosted PostgreSQL with built-in services, or Railway as an alternative.
```
***
# **UI\/UX Prototypes \(without a designer\)**
    * Use ready\-made UI libraries\:
        * **React Native Paper**\, **NativeWind**\, or **UI Kitten** for rapid styling\.
    * Develop basic screens\:
        1. Intro \+ onboarding \(minimalist\, with app goals description\)\.
        2. AI survey \(dynamic flow depending on answers\)\.
        3. Therapist recommendation \(avatar\, specialization\, short bio\)\.
        4. Chat \(WhatsApp\/Telegram style UI\, emoji support\)\.
        5. User profile \(edit\, change language\, logout\)\.
    * Ensure dark\/light theme adaptation and accessibility scaling\.
