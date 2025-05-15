# Model Integration Guide

## Overview

The Akash Chat platform is designed to be flexible and extensible, allowing for the integration of various AI language models. This guide provides detailed instructions on how to implement new models into the platform.

## Model Integration Architecture

The platform uses a layered approach to model integration:

```
+----------------+     +----------------+     +----------------+
|                |     |                |     |                |
|  Frontend      | --> |  Backend API   | --> |  LLM Gateway   |
|  Model Config  |     |  Routes        |     |  Provider      |
|                |     |                |     |  System        |
+----------------+     +----------------+     +----------------+
                                                    |
                                                    v
                                              +-----------+
                                              |           |
                                              |  Model    |
                                              |  Endpoint |
                                              |           |
                                              +-----------+
```

### Key Components in Model Integration

1. **Frontend Model Configuration**
   - Defined in `app/config/models.ts`
   - Specifies model metadata, parameters, and UI elements

2. **LLM Gateway Provider System**
   - Located in `akash-llm-gateway/src/services/providers`
   - Handles routing to appropriate model endpoints
   - Manages multiple endpoints for load balancing

3. **Model Endpoints**
   - External services that implement the OpenAI-compatible API
   - Can be deployed on Akash Network or other providers

## Step-by-Step Implementation Guide

### 1. Define the Model in Frontend

Add a new model entry to the `models` array in `app/config/models.ts`:

```typescript
{
  id: 'new-model-id',               // Unique identifier for the model
  name: 'New Model Name',            // Display name
  description: 'Description of capabilities',  // Brief description
  available: true,                   // Availability flag
  temperature: 0.7,                  // Default temperature
  top_p: 0.9,                        // Default top_p
  tokenLimit: 32000,                 // Context window size
  parameters: '7B',                  // Parameter count
  architecture: 'Transformer',       // Model architecture
  hf_repo: 'org/model-repo-name',    // HuggingFace repo if applicable
  aboutContent: `Detailed description of the model's capabilities...`, // Longer description
  infoContent: `
  * ⚡ Key feature 1
  * 🧠 Key feature 2
  * 🌐 Key feature 3
  * 🔍 Key feature 4`,              // Bullet points for UI
  thumbnailId: 'appropriate-thumbnail', // Reference to a thumbnail icon
  deployUrl: 'https://console.akash.network/path-to-deployment-template' // Deployment URL
}
```

### 2. Create a Thumbnail (Optional)

If your model needs a custom thumbnail:

1. Create a new SVG component in `components/branding/`
2. Export the component and reference it in `components/branding/model-thumbnail.tsx`

Example SVG component (`components/branding/new-model.tsx`):

```tsx
export const NewModelIcon = ({ className = "" }: { className?: string }) => (
  <svg
    className={className}
    width="32"
    height="32"
    viewBox="0 0 32 32"
    xmlns="http://www.w3.org/2000/svg"
  >
    {/* SVG content here */}
  </svg>
);
```

Update the `ModelThumbnail` component to include your new icon:

```tsx
import { NewModelIcon } from './new-model';

// Inside the component
case 'new-model-id':
  return <NewModelIcon className={className} />;
```

### 3. Configure LLM Gateway Endpoints

Update the LLM Gateway configuration to include endpoints for your new model. This is done through environment variables.

#### Environment Configuration

Add the following to your `.env` file in the LLM Gateway directory:

```
ENDPOINTS=[{"url":"https://your-model-endpoint.com","apiKey":"your-api-key","weight":1,"models":["new-model-id"]}]
```

For multiple endpoints with load balancing:

```
ENDPOINTS=[
  {"url":"https://endpoint1.com","apiKey":"key1","weight":2,"models":["new-model-id"]},
  {"url":"https://endpoint2.com","apiKey":"key2","weight":1,"models":["new-model-id"]}
]
```

### 4. Implement the Model Endpoint

Your model endpoint must implement an OpenAI-compatible API. Key requirements:

#### API Compatibility Requirements

1. **Chat Completions Endpoint**
   - Implement `POST /v1/chat/completions`
   - Accept input in the format:

```json
{
  "model": "new-model-id",
  "messages": [
    {"role": "system", "content": "System instruction"},
    {"role": "user", "content": "User message"}
  ],
  "temperature": 0.7,
  "top_p": 0.9,
  "stream": true
}
```

2. **Response Format**
   - For non-streaming responses:

```json
{
  "id": "chatcmpl-123",
  "object": "chat.completion",
  "created": 1677858242,
  "model": "new-model-id",
  "choices": [{
    "index": 0,
    "message": {
      "role": "assistant",
      "content": "Response content"
    },
    "finish_reason": "stop"
  }],
  "usage": {
    "prompt_tokens": 13,
    "completion_tokens": 7,
    "total_tokens": 20
  }
}
```

   - For streaming responses, implement server-sent events with chunks in the format:

```
data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1677858242,"model":"new-model-id","choices":[{"index":0,"delta":{"content":"H"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1677858242,"model":"new-model-id","choices":[{"index":0,"delta":{"content":"ello"},"finish_reason":null}]}

data: {"id":"chatcmpl-123","object":"chat.completion.chunk","created":1677858242,"model":"new-model-id","choices":[{"index":0,"delta":{"content":""},"finish_reason":"stop"}]}

data: [DONE]
```

### 5. Testing Your Model Integration

To test your model integration:

1. **Start the LLM Gateway**
   - Update environment variables
   - Run the gateway service

2. **Start the Frontend Application**
   - Ensure it's configured to connect to your LLM Gateway

3. **Test Basic Functionality**
   - Select your model in the UI
   - Send test messages
   - Verify responses

4. **Test Edge Cases**
   - Long messages
   - Code generation
   - Special characters
   - Streaming vs. non-streaming

### 6. Deploying Your Model

To deploy your model on the Akash Network:

1. **Prepare a Docker Container**
   - Package your model and API server
   - Ensure it implements the OpenAI-compatible API

2. **Create Deployment Configuration**
   - Create an SDL file for Akash deployment
   - Example in the `deploy.yml` file

3. **Deploy Using Akash Console**
   - Use the Akash Console to deploy your container
   - Connect it to your LLM Gateway

## Advanced Configuration

### Weighted Load Balancing

The LLM Gateway supports weighted load balancing across multiple endpoints for the same model. This is configured through the `weight` parameter in the endpoint configuration.

### Model-Specific Parameters

You can customize model behavior by adjusting parameters in the model configuration:

```typescript
{
  // Standard parameters
  id: 'new-model-id',
  // ...
  
  // Custom parameters
  contextWindowMultiplier: 1.5,  // For models with different token counting methods
  apiCompatibility: 'openai-v1',  // API compatibility version
  customHeaders: {               // Custom headers for API requests
    'X-Custom-Header': 'value'
  }
}
```

### Fallback Configuration

You can configure fallback behavior for when a model endpoint is unavailable:

```
FALLBACK_MODEL=fallback-model-id
RETRY_ATTEMPTS=3
RETRY_DELAY=1000
```

## Troubleshooting

### Common Issues

1. **Model Not Appearing in UI**
   - Verify model configuration in `models.ts`
   - Check `available` flag is set to `true`

2. **Connection Errors**
   - Verify endpoint URL and API key
   - Check network connectivity
   - Ensure endpoint implements the correct API format

3. **Streaming Issues**
   - Verify endpoint correctly implements SSE format
   - Check headers include `'Content-Type': 'text/event-stream'`

4. **Performance Problems**
   - Consider adding more endpoints for load balancing
   - Adjust weights based on endpoint capacity

### Debugging Tools

1. **LLM Gateway Logs**
   - Enable debug logging with `DEBUG=true`
   - Check logs for connection and parsing errors

2. **API Testing**
   - Use tools like Postman to test endpoint directly
   - Verify request/response format matches OpenAI standard

## Example Implementation

### Complete Model Configuration Example

```typescript
// In app/config/models.ts

export const models = [
  // ... existing models
  {
    id: 'llama3-70b',
    name: 'Llama 3 70B',
    description: 'Meta\'s advanced large language model with 70B parameters',
    available: true,
    temperature: 0.7,
    top_p: 0.9,
    tokenLimit: 32000,
    parameters: '70B',
    architecture: 'Transformer',
    hf_repo: 'meta-llama/Llama-3-70b',
    aboutContent: `Llama 3 70B is Meta's most advanced large language model, featuring 70 billion parameters. It delivers exceptional performance across various tasks including reasoning, coding, and instruction following.`,
    infoContent: `
    * ⚡ Advanced reasoning and problem-solving
    * 🧠 Strong coding capabilities across multiple languages
    * 🌐 Comprehensive knowledge base through 2023
    * 🔍 Improved context understanding and handling`,
    thumbnailId: 'llama-3',
    deployUrl: 'https://console.akash.network/new-deployment?template=llama3-70b'
  }
];
```

### Custom Endpoint Implementation Example

A basic Node.js server implementing the OpenAI-compatible API for a custom model:

```javascript
const express = require('express');
const cors = require('cors');
const bodyParser = require('body-parser');

const app = express();
app.use(cors());
app.use(bodyParser.json());

// Authentication middleware
app.use((req, res, next) => {
  const authHeader = req.headers.authorization;
  if (!authHeader || !authHeader.startsWith('Bearer ')) {
    return res.status(401).json({ error: 'Unauthorized' });
  }
  // Validate API key (example implementation)
  const apiKey = authHeader.split(' ')[1];
  if (apiKey !== process.env.API_KEY) {
    return res.status(401).json({ error: 'Invalid API key' });
  }
  next();
});

// Chat completions endpoint
app.post('/v1/chat/completions', async (req, res) => {
  const { messages, model, temperature, stream } = req.body;
  
  if (stream) {
    // Set headers for streaming response
    res.setHeader('Content-Type', 'text/event-stream');
    res.setHeader('Cache-Control', 'no-cache');
    res.setHeader('Connection', 'keep-alive');
    
    // Process messages and generate response in chunks
    // This is where you'd implement your actual model inference
    const response = "This is a sample response from the model.";
    
    // Send response in chunks
    for (let i = 0; i < response.length; i += 5) {
      const chunk = response.substring(i, Math.min(i + 5, response.length));
      const data = {
        id: `chatcmpl-${Date.now()}`,
        object: 'chat.completion.chunk',
        created: Math.floor(Date.now() / 1000),
        model,
        choices: [{
          index: 0,
          delta: { content: chunk },
          finish_reason: i + 5 >= response.length ? 'stop' : null
        }]
      };
      
      res.write(`data: ${JSON.stringify(data)}\n\n`);
      await new Promise(resolve => setTimeout(resolve, 100)); // Simulate delay
    }
    
    res.write('data: [DONE]\n\n');
    res.end();
  } else {
    // Non-streaming response
    // Process messages and generate complete response
    const response = "This is a sample response from the model.";
    
    res.json({
      id: `chatcmpl-${Date.now()}`,
      object: 'chat.completion',
      created: Math.floor(Date.now() / 1000),
      model,
      choices: [{
        index: 0,
        message: {
          role: 'assistant',
          content: response
        },
        finish_reason: 'stop'
      }],
      usage: {
        prompt_tokens: 10,
        completion_tokens: response.length / 4, // Approximate
        total_tokens: 10 + (response.length / 4)
      }
    });
  }
});

// Models endpoint
app.get('/v1/models', (req, res) => {
  res.json({
    object: 'list',
    data: [
      {
        id: 'my-custom-model',
        object: 'model',
        created: Math.floor(Date.now() / 1000),
        owned_by: 'organization-owner'
      }
    ]
  });
});

const PORT = process.env.PORT || 3000;
app.listen(PORT, () => {
  console.log(`Server running on port ${PORT}`);
});
```

## Conclusion

Implementing new models in the Akash Chat platform involves defining the model in the frontend configuration and setting up compatible endpoints in the LLM Gateway. By following this guide, you can extend the platform with custom models while maintaining compatibility with the existing architecture.

For more detailed information about specific components, refer to the following documentation:

- [Frontend Documentation](./frontend.md) for details on the UI implementation
- [LLM Gateway Documentation](./llm-gateway.md) for details on the backend services
- [Deployment Guide](./deployment.md) for information on deploying your models on Akash Network