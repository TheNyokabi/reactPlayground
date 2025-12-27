# React + TypeScript + Vite

This template provides a minimal setup to get React working in Vite with HMR and some ESLint rules.

Currently, two official plugins are available:

- [@vitejs/plugin-react](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react) uses [Babel](https://babeljs.io/) for Fast Refresh
- [@vitejs/plugin-react-swc](https://github.com/vitejs/vite-plugin-react/blob/main/packages/plugin-react-swc) uses [SWC](https://swc.rs/) for Fast Refresh

## Integrating Large Language Models (LLMs)

This playground can be extended to integrate with various LLM APIs. Here's how to add LLM capabilities to your React application.

### Supported LLM Providers

- **OpenAI** (GPT-4, GPT-3.5-turbo)
- **Anthropic** (Claude)
- **Google AI** (Gemini)
- **Cohere**
- **HuggingFace** (Open-source models)

### Installation

Install the necessary dependencies:

```bash
# For OpenAI
npm install openai

# For Anthropic Claude
npm install @anthropic-ai/sdk

# For Google Gemini
npm install @google/generative-ai
```

### Basic Implementation Example

Create a new component `src/components/LLMChat.tsx`:

```tsx
import { useState } from 'react';

function LLMChat() {
  const [input, setInput] = useState('');
  const [messages, setMessages] = useState<Array<{role: string, content: string}>>([]);
  const [loading, setLoading] = useState(false);

  const sendMessage = async () => {
    if (!input.trim()) return;

    setLoading(true);
    const userMessage = { role: 'user', content: input };
    setMessages(prev => [...prev, userMessage]);

    try {
      // Call your backend API that handles LLM requests
      const response = await fetch('/api/chat', {
        method: 'POST',
        headers: { 'Content-Type': 'application/json' },
        body: JSON.stringify({ message: input }),
      });

      const data = await response.json();
      setMessages(prev => [...prev, { role: 'assistant', content: data.response }]);
    } catch (error) {
      console.error('Error:', error);
    } finally {
      setLoading(false);
      setInput('');
    }
  };

  return (
    <div>
      <div className="messages">
        {messages.map((msg, idx) => (
          <div key={idx} className={msg.role}>
            {msg.content}
          </div>
        ))}
      </div>
      <input
        value={input}
        onChange={(e) => setInput(e.target.value)}
        onKeyDown={(e) => e.key === 'Enter' && sendMessage()}
        disabled={loading}
      />
      <button onClick={sendMessage} disabled={loading}>
        {loading ? 'Sending...' : 'Send'}
      </button>
    </div>
  );
}

export default LLMChat;
```

### Security Best Practices

**⚠️ IMPORTANT: Never expose API keys in frontend code!**

1. **Use Environment Variables**: Store API keys in `.env` files (never commit these)
   ```
   VITE_API_URL=http://localhost:3000
   ```

2. **Backend Proxy**: Always use a backend server to make LLM API calls
   - Your React app calls your backend
   - Your backend securely stores API keys
   - Your backend makes requests to LLM providers

3. **Rate Limiting**: Implement rate limiting to prevent abuse

4. **Input Validation**: Sanitize user inputs before sending to LLM APIs

### Backend API Example (Node.js/Express)

```javascript
// server.js
import express from 'express';
import OpenAI from 'openai';

const app = express();
app.use(express.json());

const openai = new OpenAI({
  apiKey: process.env.OPENAI_API_KEY
});

app.post('/api/chat', async (req, res) => {
  try {
    const { message } = req.body;
    
    const completion = await openai.chat.completions.create({
      model: "gpt-3.5-turbo",
      messages: [{ role: "user", content: message }],
    });

    res.json({ response: completion.choices[0].message.content });
  } catch (error) {
    console.error('OpenAI API error:', error);
    res.status(500).json({ error: 'Failed to process request' });
  }
});

app.listen(3000, () => console.log('Server running on port 3000'));
```

### Using OpenAI in React (via Backend)

```tsx
const callOpenAI = async (prompt: string) => {
  const response = await fetch('http://localhost:3000/api/chat', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: prompt }),
  });
  const data = await response.json();
  return data.response;
};
```

### Streaming Responses

For better UX with long responses, implement streaming:

```tsx
const streamResponse = async (prompt: string) => {
  const response = await fetch('/api/chat/stream', {
    method: 'POST',
    headers: { 'Content-Type': 'application/json' },
    body: JSON.stringify({ message: prompt }),
  });

  const reader = response.body?.getReader();
  if (!reader) return;
  
  const decoder = new TextDecoder();

  while (true) {
    const { done, value } = await reader.read();
    if (done) break;
    
    const chunk = decoder.decode(value);
    // Update UI with chunk
  }
};
```

### Common Use Cases

1. **Chatbot**: Interactive conversational interface
2. **Content Generation**: Blog posts, summaries, translations
3. **Code Assistant**: Code completion, explanation, debugging
4. **Data Analysis**: Natural language queries on data
5. **Customer Support**: Automated responses and ticket routing

### Resources

- [OpenAI API Documentation](https://platform.openai.com/docs)
- [Anthropic Claude API](https://docs.anthropic.com/claude/reference/getting-started-with-the-api)
- [Google Gemini API](https://ai.google.dev/docs)
- [LangChain for React](https://js.langchain.com/docs/get_started/introduction)

## Expanding the ESLint configuration

If you are developing a production application, we recommend updating the configuration to enable type-aware lint rules:

```js
export default tseslint.config([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      // Other configs...

      // Remove tseslint.configs.recommended and replace with this
      ...tseslint.configs.recommendedTypeChecked,
      // Alternatively, use this for stricter rules
      ...tseslint.configs.strictTypeChecked,
      // Optionally, add this for stylistic rules
      ...tseslint.configs.stylisticTypeChecked,

      // Other configs...
    ],
    languageOptions: {
      parserOptions: {
        project: ['./tsconfig.node.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
      // other options...
    },
  },
])
```

You can also install [eslint-plugin-react-x](https://github.com/Rel1cx/eslint-react/tree/main/packages/plugins/eslint-plugin-react-x) and [eslint-plugin-react-dom](https://github.com/Rel1cx/eslint-react/tree/main/packages/plugins/eslint-plugin-react-dom) for React-specific lint rules:

```js
// eslint.config.js
import reactX from 'eslint-plugin-react-x'
import reactDom from 'eslint-plugin-react-dom'

export default tseslint.config([
  globalIgnores(['dist']),
  {
    files: ['**/*.{ts,tsx}'],
    extends: [
      // Other configs...
      // Enable lint rules for React
      reactX.configs['recommended-typescript'],
      // Enable lint rules for React DOM
      reactDom.configs.recommended,
    ],
    languageOptions: {
      parserOptions: {
        project: ['./tsconfig.node.json', './tsconfig.app.json'],
        tsconfigRootDir: import.meta.dirname,
      },
      // other options...
    },
  },
])
```
