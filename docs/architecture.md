# Akash Chat Platform Architecture

## Overview

Akash Chat is a modern chat application that leverages the Akash Network's decentralized cloud infrastructure to provide access to various AI models. The platform follows a client-server architecture with a clear separation between the frontend user interface and backend AI model integration.

## System Components

### Core Components

```
+----------------+     +----------------+     +----------------+
|                |     |                |     |                |
|  Next.js       | --> |  Backend API   | --> |  LLM Gateway   |
|  Frontend      |     |  Routes        |     |  Service       |
|                |     |                |     |                |
+----------------+     +----------------+     +----------------+
                                               |       |
                                               v       v
                                       +---------------+---------------+
                                       |               |               |
                                       |  Model A      |  Model B      |
                                       |  Endpoints    |  Endpoints    |
                                       |               |               |
                                       +---------------+---------------+
```

1. **Frontend Application (Next.js)**
   - Provides the user interface for interactions
   - Handles state management and chat history
   - Renders messages with support for markdown, code blocks, etc.
   - Manages user authentication and session handling

2. **Backend API Routes**
   - Handles API requests from the frontend
   - Manages authentication and validation
   - Routes chat requests to the LLM Gateway
   - Processes transcription and file uploads

3. **LLM Gateway**
   - Manages connections to various model endpoints
   - Provides a unified interface for model access
   - Handles load balancing and failover
   - Implements model-specific configuration

4. **Model Endpoints**
   - External services that host AI models
   - Provide inference capabilities via OpenAI-compatible APIs
   - Can be distributed across multiple hosts

### Supporting Components

1. **Redis Storage**
   - Manages session data and caching
   - Potentially stores chat history

2. **Authentication System**
   - Handles user sessions and authentication
   - Manages API keys for model access

## Data Flow

1. **User Interaction Flow**
   - User enters a message in the frontend interface
   - Frontend sends the message to its API routes
   - API routes authenticate and validate the request
   - Request is forwarded to the LLM Gateway
   - Gateway selects appropriate model endpoint
   - Model generates response
   - Response flows back through the same path
   - Frontend renders the response

2. **Model Selection Flow**
   - User selects a model from the UI
   - Frontend updates state and configuration
   - Subsequent requests use the selected model
   - LLM Gateway routes to appropriate endpoints

3. **Authentication Flow**
   - User authentication handled via session routes
   - Sessions tracked in Redis store
   - API routes verify session before processing

## Component Interactions

### Frontend to Backend

The frontend communicates with the backend API routes using standard HTTP requests. The primary endpoints include:

- `/api/chat`: Handles chat messages and completions
- `/api/models`: Provides model information and configuration
- `/api/auth`: Manages user authentication
- `/api/transcription`: Handles audio transcription requests

### Backend to LLM Gateway

The backend API routes communicate with the LLM Gateway using HTTP requests to endpoints like:

- `/v1/chat/completions`: Primary endpoint for chat completions
- `/v1/models`: Endpoint to retrieve available models

### LLM Gateway to Model Endpoints

The LLM Gateway communicates with model endpoints using standardized OpenAI-compatible API calls. This allows for consistent integration regardless of the underlying model implementation.

## Technical Implementation

### Frontend (Next.js)

The frontend is built with Next.js and uses the following key technologies:

- **React**: For component-based UI development
- **TypeScript**: For type-safe code
- **Tailwind CSS**: For styling
- **Context API**: For state management

Key files and directories:

- `app/config/models.ts`: Model configuration
- `app/api/`: API route handlers
- `components/`: UI components
- `context/ChatContext.tsx`: Chat state management

### LLM Gateway

The LLM Gateway is a Node.js application with TypeScript that provides:

- **Provider System**: Extensible system for LLM providers
- **Endpoint Management**: Discovery and configuration of endpoints
- **Load Balancing**: Distribution of requests across endpoints

Key files and directories:

- `src/controllers/`: Request handlers
- `src/services/`: Business logic and provider implementations
- `src/types/`: TypeScript type definitions

## Deployment Architecture

The platform is designed to be deployed on the Akash Network's decentralized cloud infrastructure:

1. **Frontend Deployment**
   - Containerized Next.js application
   - Configured via environment variables
   - Exposed via HTTP/HTTPS

2. **LLM Gateway Deployment**
   - Containerized Node.js application
   - Configured via environment variables
   - Connected to model endpoints

3. **Model Endpoints**
   - Can be deployed on Akash or other providers
   - Must implement OpenAI-compatible API
   - Connected to the LLM Gateway

## Configuration and Customization

The platform is highly configurable through:

1. **Environment Variables**
   - Frontend: Configuration in `.env` files
   - LLM Gateway: Configuration in `.env` files

2. **Model Configuration**
   - Defined in `app/config/models.ts`
   - Customizable parameters for each model

3. **Endpoint Configuration**
   - Configured in LLM Gateway via environment variables
   - Supports multiple endpoints per model with load balancing

## Security Considerations

1. **Authentication**
   - User authentication via sessions
   - API key authentication for model access

2. **API Security**
   - HTTPS for all communications
   - Input validation and sanitization

3. **Environment Segregation**
   - Separation of frontend and backend components
   - Isolation of sensitive configuration

## Extensibility

The architecture is designed for extensibility through:

1. **Provider System**
   - Easily add new LLM providers
   - Consistent API interface

2. **Model Configuration**
   - Straightforward addition of new models
   - Flexible parameter configuration

3. **UI Components**
   - Modular design for UI customization
   - Theming support

For details on implementing new models, refer to the [Model Integration Guide](./model-integration.md).