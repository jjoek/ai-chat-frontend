# Vue.js AI Chat Platform UI

A clean, responsive AI chat interface built with Vue.js 3 and Vite. Features model switching between different AI providers with mock responses for testing.

## Features

- 🤖 **Multiple AI Models**: GPT-4, Claude, Gemini, Llama, and more
- 📱 **Responsive Design**: Works on desktop, tablet, and mobile
- 🎨 **Modern UI**: Clean interface with Tailwind CSS
- ⚡ **Fast Development**: Built with Vite for instant hot reload
- 🔄 **Model Switching**: Easy dropdown to switch between AI models
- 💬 **Rich Chat Features**: Copy messages, regenerate responses, typing indicators
- 🎯 **Mock Responses**: Realistic mock responses for each model

## Quick Start

1. **Install dependencies**:
   \`\`\`bash
   npm install
   \`\`\`

2. **Run development server**:
   \`\`\`bash
   npm run dev
   \`\`\`

3. **Open your browser**:
   Navigate to `http://localhost:3000`

## Available Scripts

- `npm run dev` - Start development server with hot reload
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally

## Project Structure

\`\`\`
├── src/
│   ├── components/
│   │   └── ChatPlatform.vue    # Main chat component
│   ├── main.js                 # App entry point
│   └── style.css              # Global styles
├── index.html                  # HTML template
├── package.json               # Dependencies
├── vite.config.js            # Vite configuration
├── tailwind.config.js        # Tailwind CSS config
└── postcss.config.js         # PostCSS config
\`\`\`

## Supported AI Models

- **OpenAI**: GPT-4, GPT-3.5 Turbo
- **Anthropic**: Claude 3 Sonnet, Claude 3 Haiku
- **Google**: Gemini Pro
- **Meta**: Llama 2 70B

## Customization

### Adding New Models

Edit the `models` array in `src/components/ChatPlatform.vue`:

\`\`\`javascript
const models = [
  {
    id: 'your-model-id',
    name: 'Your Model Name',
    provider: 'Provider Name',
    description: 'Model description',
    color: '#HEX_COLOR'
  }
]
\`\`\`

### Connecting to Real AI APIs

Replace the mock response logic in the `sendMessage()` function with actual API calls to your backend or AI service.

## Technologies

- **Vue.js 3** - Composition API
- **Vite** - Fast build tool
- **Tailwind CSS** - Utility-first CSS
- **Lucide Vue** - Beautiful icons

## License

MIT License
\`\`\`
