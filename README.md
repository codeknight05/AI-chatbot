# AI-Powered Chat Application

A production-ready, responsive AI chat application built with **React + Vite**, **Firebase Authentication/Firestore**, and the **Google Gemini API**. The application provides authenticated users with a persistent coding assistant that can generate explanations, code examples, debugging guidance, and programming help through a modern chat interface.

> **Live Application:** https://ai-chatbot-three-ruddy-70.vercel.app/
>
> **Repository:** https://github.com/codeknight05/AI-chatbot

---

## Overview

This project is a full-featured web-based AI coding assistant designed around three core ideas:

1. **Authenticated access** — users sign in before accessing private chat functionality.
2. **Persistent conversations** — chat messages are stored in Firebase Firestore and restored automatically for the authenticated user.
3. **User-provided Gemini integration** — each user supplies their own Gemini API key, which the application stores locally and uses to initialize the Gemini client.

The application is implemented as a client-side Vite application and can be deployed to static hosting platforms such as **Vercel, Netlify, or Firebase Hosting**.

---

## Key Features

### AI-Powered Programming Assistant

- Uses the **Google Gemini API** to generate conversational responses.
- Optimized system prompt instructs the model to behave as a programming assistant.
- Supports explanations of programming concepts, debugging help, algorithms, code examples, and software-development questions.
- Displays generated responses directly inside the chat conversation.

### User Authentication

- Firebase Authentication is used to manage user identity and sessions.
- Unauthenticated users cannot access the private chat interface.
- Protected routes prevent direct access to authenticated-only pages.
- Authenticated users are automatically redirected through the application setup flow.

### Persistent Chat History

- Conversations are stored in **Firebase Firestore**.
- Each message is associated with the authenticated user's Firebase UID.
- User-specific queries ensure that users load only their own messages.
- Firestore `onSnapshot` provides live updates to the conversation.
- Messages are ordered using their Firestore timestamps.
- New accounts receive an automatically generated welcome message.

### Gemini API Key Management

- Users provide their own Gemini API key through a dedicated setup screen.
- The key is stored in the browser's `localStorage` for the current implementation.
- Users are routed to the API-key setup page before entering the chat if no key is present.
- No server-side Gemini API key is required for deployment.

> **Security note:** Because the current implementation is client-side and stores the user-provided Gemini API key in browser storage, users should only provide API keys they are comfortable using in a browser-based application. A production architecture requiring stronger secret protection should move model access behind a trusted backend service.

### Modern Chat Interface

- Responsive UI designed for desktop and mobile screens.
- Separate visual treatment for user and AI messages.
- Loading indicators while Gemini generates a response.
- Markdown rendering for structured AI responses.
- Syntax highlighting for fenced code blocks.
- Support for headings, paragraphs, ordered lists, unordered lists, links, blockquotes, inline code, and code blocks.
- Dark-mode-compatible styling.
- Animated transitions and interactive UI states.

### Protected Routing

The application uses `react-router-dom` to manage application navigation.

Current routes include:

| Route | Purpose | Access |
|---|---|---|
| `/` | Landing/home page | Public |
| `/auth` | Login/authentication page | Public |
| `/api-key` | Gemini API-key setup | Authenticated |
| `/chat` | Main AI chat interface | Authenticated + API key |

The `/chat` route is protected by a reusable `PrivateRoute` component and requires both authentication and a configured Gemini API key.

---

## Architecture

The application follows a modular React architecture with clear separation between UI components, authentication/configuration utilities, routing, and external services.

```text
┌─────────────────────────────────────────────────────────────┐
│                        React + Vite                         │
│                                                             │
│  ┌────────────┐    ┌──────────────┐    ┌───────────────┐  │
│  │   Home     │    │    Login     │    │   Navigation  │  │
│  └────────────┘    └──────────────┘    └───────────────┘  │
│                                                             │
│  ┌──────────────────────┐     ┌─────────────────────────┐  │
│  │   Private Routing    │     │   API Key Setup         │  │
│  └──────────────────────┘     └─────────────────────────┘  │
│                                                             │
│  ┌───────────────────────────────────────────────────────┐  │
│  │                      Chat UI                          │  │
│  │  - Message state                                     │  │
│  │  - Firestore synchronization                          │  │
│  │  - Gemini requests                                    │  │
│  │  - Markdown/code rendering                            │  │
│  └───────────────────────────────────────────────────────┘  │
└───────────────────────┬─────────────────┬─────────────────┘
                        │                 │
                        ▼                 ▼
               ┌────────────────┐  ┌────────────────────┐
               │ Firebase Auth  │  │ Firebase Firestore │
               └────────────────┘  └────────────────────┘
                                          │
                                          │ user-scoped
                                          │ messages
                                          ▼
                               ┌──────────────────────┐
                               │   Persistent Chat    │
                               │       History        │
                               └──────────────────────┘

                        ┌──────────────────────────┐
                        │      Gemini API          │
                        │ @google/generative-ai   │
                        └────────────▲─────────────┘
                                     │
                           user-provided API key
                                     │
                              browser localStorage
```

---

## Application Flow

### 1. User opens the application

The home page is publicly accessible. The application initializes Firebase and checks the current authentication state.

### 2. User authentication

A user signs in through the authentication flow. Once Firebase confirms the session, protected routes become accessible.

### 3. API key configuration

After authentication, the application checks for a `gemini_api_key` entry in browser `localStorage`.

- If the key is missing, the user is redirected to `/api-key`.
- If the key already exists, the user can proceed directly to `/chat`.

### 4. Chat initialization

When the chat page loads:

- The authenticated user's Firebase UID is used to query Firestore.
- Existing messages belonging to that user are loaded.
- If no messages exist, a welcome assistant message is created.
- A Firestore real-time listener (`onSnapshot`) keeps the UI synchronized with stored messages.

### 5. Sending a message

When the user submits a prompt:

1. The message is validated to ensure it is not empty.
2. The user's message is written to Firestore.
3. The Gemini client is initialized using the stored API key.
4. The application sends a programming-assistant prompt plus the user's question to Gemini.
5. The model response is extracted from the Gemini result.
6. The AI response is persisted to Firestore and associated with the originating user message.
7. The UI updates automatically through the Firestore listener.

### 6. Response rendering

AI responses are rendered with `react-markdown` and enhanced with syntax highlighting for fenced code blocks using `react-syntax-highlighter`.

---

## Data Model

The chat application stores messages in a Firestore `messages` collection.

A message document follows the general structure:

```text
messages/{messageId}
│
├── text: string
├── timestamp: Firestore Timestamp
├── role: "user" | "ai"
├── userId: string
├── isWelcomeMessage: boolean (optional)
└── replyTo: string (optional)
```

### Important fields

- **`text`** — message or generated response content.
- **`timestamp`** — Firestore server timestamp used for chronological ordering.
- **`role`** — identifies whether the message came from the user or the AI assistant.
- **`userId`** — associates the message with the authenticated Firebase user.
- **`isWelcomeMessage`** — identifies automatically generated initial assistant content.
- **`replyTo`** — links an AI response to the user message that triggered it.

This user-scoped structure allows the client to query and render a user's own conversation history.

---

## Technology Stack

### Frontend

- **React 18** — component-based UI architecture.
- **Vite** — development server and production build tooling.
- **JavaScript / JSX** — application logic and UI components.
- **React Router** — client-side routing and protected navigation.
- **Tailwind CSS** — utility-first responsive styling.
- **Framer Motion** — UI animation support.

### Authentication & Data

- **Firebase Authentication** — user authentication and session management.
- **Firebase Firestore** — persistent user-scoped chat history and real-time synchronization.
- **react-firebase-hooks** — React-friendly Firebase authentication state handling.

### AI

- **Google Gemini API** — conversational AI and programming assistance.
- **`@google/generative-ai`** — JavaScript SDK used to initialize and communicate with the Gemini model.

### Content Rendering

- **react-markdown** — Markdown rendering for AI responses.
- **react-syntax-highlighter** — syntax highlighting for generated code.

### Forms & Validation

- **Formik** — form state and submission management.
- **Yup** — client-side validation schemas.

### Quality & Tooling

- **ESLint** — static code analysis and linting.
- **PostCSS / Autoprefixer** — CSS processing.
- **npm** — dependency and script management.

---

## Project Structure

```text
AI-chatbot/
│
├── .github/
│   └── workflows/
│       └── deploy-pages.yml
│
├── public/
│   ├── _redirects
│   └── vite.svg
│
├── src/
│   ├── assets/
│   │   └── react.svg
│   │
│   ├── components/
│   │   ├── ApiKeyPrompt.jsx
│   │   ├── Chat.jsx
│   │   ├── Home.jsx
│   │   ├── Login.jsx
│   │   ├── Navigation.jsx
│   │   └── PrivateRoute.jsx
│   │
│   ├── config/
│   │   ├── firebase.js
│   │   └── gemini.js
│   │
│   ├── utils/
│   │   ├── auth.js
│   │   └── index.css
│   │
│   ├── App.jsx
│   └── main.jsx
│
├── .gitignore
├── eslint.config.js
├── index.html
├── netlify.toml
├── package.json
├── package-lock.json
├── postcss.config.js
├── tailwind.config.js
├── vercel.json
└── vite.config.js
```

### Component responsibilities

#### `App.jsx`

The application root configures routing, authentication-aware navigation, and the conditions required before entering the chat page.

#### `Home.jsx`

Provides the public landing page and introduction to the application.

#### `Login.jsx`

Handles the user authentication interface.

#### `PrivateRoute.jsx`

Restricts protected application routes to authenticated users.

#### `ApiKeyPrompt.jsx`

Provides the Gemini API key setup flow required before the chat interface becomes available.

#### `Chat.jsx`

Contains the primary chat experience, including:

- message state management
- Firestore reads/writes
- real-time Firestore subscriptions
- Gemini API requests
- loading/error states
- Markdown rendering
- syntax highlighting

#### `Navigation.jsx`

Provides application-level navigation and authentication-aware UI.

#### `firebase.js`

Initializes the Firebase application, authentication service, and Firestore client.

#### `gemini.js`

Provides Gemini model initialization using the locally configured user API key.

---

## Installation

### Prerequisites

Make sure the following are installed:

- **Node.js**
- **npm**
- A Firebase project
- A Google Gemini API key

### Clone the repository

```bash
git clone https://github.com/codeknight05/AI-chatbot.git
cd AI-chatbot
```

### Install dependencies

```bash
npm install
```

### Start the development server

```bash
npm run dev
```

Vite will start the local development server and provide the local application URL in the terminal.

---

## Build for Production

Create an optimized production build with:

```bash
npm run build
```

The production output is generated in:

```text
dist/
```

To preview the production build locally:

```bash
npm run preview
```

---

## Linting

Run the ESLint configuration with:

```bash
npm run lint
```

The project uses ESLint along with React and React Hooks plugins to identify common JavaScript and React issues.

---

## Firebase Configuration

Create/configure a Firebase project with the services required by the application.

### Authentication

Enable the desired sign-in provider in Firebase Authentication.

The application uses Firebase Auth to:

- authenticate users
- maintain sessions
- expose the authenticated Firebase UID
- protect private routes

After deploying the application, add the production domain to the Firebase Authentication **authorized domains** list.

### Firestore

Create a Firestore database for persistent chat history.

The application stores chat messages in a `messages` collection and associates each document with the authenticated user's UID.

For production use, Firestore security rules should enforce user-level access so that users can only read and write their own documents.

A typical rule design should conceptually enforce:

```text
request.auth != null
AND request.auth.uid == resource.data.userId
```

The exact rules should be tailored to the project's deployment and Firestore document lifecycle.

---

## Gemini API Configuration

This project intentionally uses a **user-provided Gemini API key**.

The current application flow is:

```text
User enters Gemini API key
        ↓
API key stored in browser localStorage
        ↓
Chat page initializes Gemini client
        ↓
Gemini model receives user prompt
        ↓
Generated response returned to browser
        ↓
Response persisted to Firestore
```

This means the deployed application does **not** require a Gemini secret in the Vercel environment for the current implementation.

### Important security consideration

Because the Gemini API key is used from client-side code and stored in browser storage, this design should be treated as a client-side convenience architecture rather than a high-security secret-management architecture.

For a stronger production implementation, the recommended architecture would be:

```text
React Client
    ↓
Authenticated Backend API
    ↓
Gemini API
```

The backend would hold the provider credential and prevent users from exposing their own or an application-wide provider key to the browser.

---

## Deployment

The application is a static Vite build and is suitable for modern static hosting platforms.

### Vercel

Recommended Vercel settings:

```text
Framework Preset: Vite
Build Command: npm run build
Output Directory: dist
Install Command: npm install
```

Production deployment URL:

**https://ai-chatbot-three-ruddy-70.vercel.app/**

After deployment, configure the production domain in Firebase Authentication's authorized domains.

### Netlify

Recommended settings:

```text
Build Command: npm run build
Publish Directory: dist
```

A `netlify.toml` file is included in the repository for deployment configuration.

### Firebase Hosting

The Vite-generated `dist` directory can also be served using Firebase Hosting.

---

## User Experience

The application is designed around a simple flow:

```text
Landing Page
    ↓
Authentication
    ↓
Gemini API Key Setup
    ↓
AI Chat
    ↓
Persistent Conversation History
```

The chat interface includes:

- responsive message containers
- visual separation between user and assistant messages
- loading animation while waiting for Gemini
- Markdown support
- syntax-highlighted code blocks
- links that open safely in a new tab
- dark-mode-compatible styling
- animated UI transitions

---

## Error Handling

The chat flow includes basic runtime error handling for failed Gemini requests and other asynchronous operations.

When an AI request fails:

1. The error is logged to the browser console.
2. An error message is inserted into the conversation UI.
3. The loading state is cleared.
4. The message input is reset so the user can attempt another request.

Firestore subscription errors are also handled without leaving the UI permanently blocked.

---

## Design Decisions

### Why React + Vite?

React provides a modular component model suited to interactive chat interfaces, while Vite provides fast local development and efficient production builds.

### Why Firebase?

Firebase provides managed authentication and a hosted document database without requiring the project to maintain a custom backend for user management and message persistence.

### Why Firestore?

Firestore's document model maps naturally to chat messages, while real-time listeners make it possible to synchronize the user interface with stored conversation data.

### Why Gemini?

Gemini provides a generative AI API that can produce natural-language explanations and code-oriented responses suitable for a programming assistant.

### Why user-provided API keys?

The current implementation avoids requiring the deployed frontend to contain a shared application-wide Gemini credential. Each user supplies their own API key instead.

---

## Limitations

The current version is intentionally a client-side architecture and therefore has several limitations:

- Gemini API keys are stored in browser `localStorage`.
- The Gemini request is performed from the client.
- There is no dedicated Node.js/Express backend in the current implementation.
- Conversation memory is represented by persisted Firestore messages rather than server-side conversational state.
- Production Firestore security rules must be configured correctly to enforce user isolation.

These trade-offs keep the application simple to deploy while leaving a clear path for a future backend architecture.

---

## Future Improvements

Potential improvements include:

- Add a **Node.js/Express backend** for Gemini requests.
- Move Gemini credentials to secure server-side environment variables.
- Implement stronger Firestore security rules and validation.
- Add conversation/session management so users can create and switch between multiple chats.
- Add message editing, regeneration, and deletion.
- Add streaming Gemini responses for lower perceived latency.
- Add retry and exponential backoff handling for transient API failures.
- Add usage monitoring and rate limiting.
- Add automated unit and integration tests.
- Add CI checks for linting and production builds.
- Add richer model-selection settings.

---

## Development Scripts

| Command | Purpose |
|---|---|
| `npm run dev` | Start Vite development server |
| `npm run build` | Build production assets |
| `npm run preview` | Preview production build locally |
| `npm run lint` | Run ESLint |

---

## Dependencies

The project uses a modern frontend dependency set including:

- `react`
- `react-dom`
- `react-router-dom`
- `firebase`
- `react-firebase-hooks`
- `@google/generative-ai`
- `react-markdown`
- `react-syntax-highlighter`
- `formik`
- `yup`
- `framer-motion`
- `tailwindcss`
- `@heroicons/react`
- `react-icons`

The complete dependency versions are defined in `package.json` and locked through `package-lock.json`.

---

## Engineering Highlights

This project demonstrates practical implementation of:

- component-based frontend architecture
- client-side routing
- protected routes
- authentication state management
- cloud database integration
- real-time data synchronization
- third-party API integration
- asynchronous JavaScript workflows
- browser-side configuration management
- Markdown and code rendering
- responsive UI development
- production builds and static deployment
- linting and frontend tooling

---

## Repository

**GitHub:** https://github.com/codeknight05/AI-chatbot

**Live:** https://ai-chatbot-three-ruddy-70.vercel.app/

---

## License

No explicit open-source license is currently specified in the repository. Treat the project as source-available under the repository owner's rights unless a license is added.
