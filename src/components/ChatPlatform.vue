<template>
  <div v-if="models.length <= 0">Loading ...</div>
  <div v-else class="flex flex-col h-screen bg-gray-50">
    <!-- Header -->
    <header class="bg-white border-b border-gray-200 px-4 py-3 shadow-sm">
      <div class="flex items-center justify-between max-w-6xl mx-auto">
        <div class="flex items-center space-x-3">
          <div
            class="w-10 h-10 bg-gradient-to-br from-blue-500 to-purple-600 rounded-lg flex items-center justify-center"
          >
            <MessageCircleIcon class="w-6 h-6 text-white" />
          </div>
          <div>
            <h1 class="text-xl font-bold text-gray-900">JJoek AI Chat</h1>
            <p class="text-sm text-gray-500">
              Choose your AI model and start chatting
            </p>
          </div>
        </div>

        <!-- Model Selector -->
        <div class="relative">
          <button
            @click="isModelDropdownOpen = !isModelDropdownOpen"
            class="flex items-center space-x-3 bg-gray-50 hover:bg-gray-100 px-4 py-2 rounded-xl border border-gray-200 transition-all duration-200"
          >
            <div
              class="w-8 h-8 rounded-lg flex items-center justify-center text-white text-sm font-medium"
              :style="{ backgroundColor: currentModel.color }"
            >
              {{ currentModel.name.charAt(0) }}
            </div>
            <div class="text-left">
              <div class="text-sm font-medium text-gray-900">
                {{ currentModel.name }}
              </div>
              <div class="text-xs text-gray-500">
                {{ currentModel.provider }}
              </div>
            </div>
            <ChevronDownIcon class="w-4 h-4 text-gray-500" />
          </button>

          <!-- Dropdown Menu -->
          <div
            v-if="isModelDropdownOpen"
            class="absolute right-0 mt-2 w-72 bg-white border border-gray-200 rounded-xl shadow-lg z-50 py-2"
          >
            <div
              class="px-3 py-2 text-xs font-medium text-gray-500 uppercase tracking-wide"
            >
              Available Models
            </div>
            <button
              v-for="model in models"
              :key="model.id"
              @click="selectModel(model)"
              class="w-full text-left px-3 py-2 hover:bg-gray-50 flex items-center space-x-3 transition-colors"
              :class="{ 'bg-blue-50': selectedModelId === model.id }"
            >
              <div
                class="w-8 h-8 rounded-lg flex items-center justify-center text-white text-sm font-medium"
                :style="{ backgroundColor: model.color }"
              >
                {{ model.name.charAt(0) }}
              </div>
              <div class="flex-1">
                <div class="text-sm font-medium text-gray-900">
                  {{ model.name }}
                </div>
                <div class="text-xs text-gray-500">
                  {{ model.provider }} • {{ model.description }}
                </div>
              </div>
              <div
                v-if="selectedModelId === model.id"
                class="w-2 h-2 bg-blue-500 rounded-full"
              ></div>
            </button>
          </div>
        </div>
      </div>
    </header>

    <!-- Chat Area -->
    <div class="flex-1 overflow-hidden">
      <div class="h-full max-w-4xl mx-auto flex flex-col">
        <!-- Messages Container -->
        <div
          ref="messagesContainer"
          class="flex-1 overflow-y-auto custom-scrollbar px-4 py-6 space-y-6"
        >
          <!-- Welcome Message -->
          <div v-if="messages.length === 0" class="text-center py-12">
            <div
              class="w-16 h-16 bg-gradient-to-br from-blue-500 to-purple-600 rounded-2xl flex items-center justify-center mx-auto mb-4"
            >
              <SparklesIcon class="w-8 h-8 text-white" />
            </div>
            <h3 class="text-xl font-semibold text-gray-900 mb-2">
              Welcome to AI Chat Platform
            </h3>
            <p class="text-gray-500 mb-4">
              Start a conversation with {{ currentModel.name }}
            </p>
            <div class="flex flex-wrap justify-center gap-2">
              <button
                v-for="suggestion in suggestions"
                :key="suggestion"
                @click="sendMessage(suggestion)"
                class="px-4 py-2 bg-white border border-gray-200 rounded-full text-sm text-gray-700 hover:bg-gray-50 transition-colors"
              >
                {{ suggestion }}
              </button>
            </div>
          </div>

          <!-- Chat Messages -->
          <div
            v-for="message in messages"
            :key="message.id"
            class="flex items-start space-x-3"
            :class="{
              'flex-row-reverse space-x-reverse': message.role === 'user',
            }"
          >
            <!-- Avatar -->
            <div class="flex-shrink-0">
              <div
                v-if="message.role === 'user'"
                class="w-8 h-8 bg-blue-500 rounded-full flex items-center justify-center"
              >
                <UserIcon class="w-4 h-4 text-white" />
              </div>
              <div
                v-else
                class="w-8 h-8 rounded-full flex items-center justify-center text-white text-xs font-medium"
                :style="{ backgroundColor: currentModel.color }"
              >
                {{ currentModel.name.charAt(0) }}
              </div>
            </div>

            <!-- Message Content -->
            <div class="flex-1 max-w-3xl">
              <div
                class="px-4 py-3 rounded-2xl relative group"
                :class="
                  message.role === 'user'
                    ? 'bg-blue-500 text-white ml-12'
                    : 'bg-white border border-gray-200 text-gray-900 mr-12'
                "
              >
                <vue-markdown :source="message.content" />

                <!-- Message Actions -->
                <div
                  v-if="message.role === 'assistant'"
                  class="absolute -bottom-8 right-0 flex items-center space-x-2 opacity-0 group-hover:opacity-100 transition-opacity"
                >
                  <button
                    @click="copyMessage(message.content, message.id)"
                    class="p-1 hover:bg-gray-100 rounded text-gray-500 hover:text-gray-700"
                    :title="
                      copiedMessageId === message.id
                        ? 'Copied!'
                        : 'Copy message'
                    "
                  >
                    <CheckIcon
                      v-if="copiedMessageId === message.id"
                      class="w-4 h-4 text-green-500"
                    />
                    <CopyIcon v-else class="w-4 h-4" />
                  </button>
                  <button
                    @click="regenerateResponse(message.id)"
                    class="p-1 hover:bg-gray-100 rounded text-gray-500 hover:text-gray-700"
                    title="Regenerate response"
                  >
                    <RefreshCwIcon class="w-4 h-4" />
                  </button>
                </div>
              </div>

              <!-- Message Info -->
              <div
                class="text-xs text-gray-500 mt-1 px-1"
                :class="{ 'text-right': message.role === 'user' }"
              >
                {{ message.role === "user" ? "You" : message.modelName }} •
                {{ formatTime(message.timestamp) }}
              </div>
            </div>
          </div>

          <!-- Typing Indicator -->
          <div v-if="isTyping" class="flex items-start space-x-3">
            <div
              class="w-8 h-8 rounded-full flex items-center justify-center text-white text-xs font-medium"
              :style="{ backgroundColor: currentModel.color }"
            >
              {{ currentModel.name.charAt(0) }}
            </div>
            <div
              class="bg-white border border-gray-200 px-4 py-3 rounded-2xl mr-12"
            >
              <div class="flex space-x-1">
                <div class="w-2 h-2 bg-gray-400 rounded-full typing-dot"></div>
                <div class="w-2 h-2 bg-gray-400 rounded-full typing-dot"></div>
                <div class="w-2 h-2 bg-gray-400 rounded-full typing-dot"></div>
              </div>
            </div>
          </div>
        </div>

        <!-- Input Area -->
        <div class="px-4 py-4">
          <form @submit.prevent="handleSubmit" class="flex items-end space-x-3">
            <div class="flex-1 relative">
              <textarea
                ref="messageInput"
                v-model="inputMessage"
                placeholder="Type your message..."
                rows="1"
                class="w-full px-4 py-3 pr-12 border border-gray-300 rounded-2xl focus:outline-none focus:ring-2 focus:ring-blue-500 focus:border-transparent resize-none"
                style="min-height: 48px; max-height: 120px"
                @keydown.enter.exact.prevent="handleSubmit"
                @input="autoResize"
                :disabled="isTyping"
              ></textarea>

              <!-- Character Count -->
              <div
                v-if="inputMessage.length > 0"
                class="absolute bottom-2 right-2 mb-4 mr-4 text-xs text-gray-400"
              >
                {{ inputMessage.length }}
              </div>
            </div>

            <div class="flex mb-3">
              <!-- Clear Chat -->
              <button
                v-if="messages.length > 0"
                @click="clearChat"
                type="button"
                class="p-3 text-gray-500 hover:text-red-500 hover:bg-red-50 rounded-full transition-colors"
                title="Clear chat"
              >
                <Trash2Icon class="w-5 h-5" />
              </button>

              <!-- Send Button -->
              <button
                type="submit"
                :disabled="!inputMessage.trim() || isTyping"
                class="p-3 bg-blue-500 text-white rounded-full hover:bg-blue-600 disabled:opacity-50 disabled:cursor-not-allowed transition-colors"
                title="Send message"
              >
                <SendIcon class="w-5 h-5" />
              </button>
            </div>
          </form>
        </div>
      </div>
    </div>

    <!-- Click outside to close dropdown -->
    <div
      v-if="isModelDropdownOpen"
      @click="isModelDropdownOpen = false"
      class="fixed inset-0 z-40"
    ></div>
  </div>
</template>

<script setup>
import axios from "axios";
import { ref, computed, nextTick, onMounted } from "vue";
import VueMarkdown from "vue-markdown-render";
import {
  MessageCircleIcon,
  ChevronDownIcon,
  UserIcon,
  SparklesIcon,
  CopyIcon,
  CheckIcon,
  RefreshCwIcon,
  SendIcon,
  Trash2Icon,
} from "lucide-vue-next";
const apiUrl = import.meta.env.VITE_API_URL;

// Available AI models
let models = ref([]);
let currentModel = ref({});

// Reactive state
const selectedModelId = ref("gpt-4");
const isModelDropdownOpen = ref(false);
const inputMessage = ref("");
const messages = ref([]);
const isTyping = ref(false);
const copiedMessageId = ref(null);
const messagesContainer = ref(null);
const messageInput = ref(null);

// Sample suggestions
const suggestions = [
  "What can you help me with?",
  "Explain quantum computing",
  "Write a creative story",
  "Help me debug code",
];

// Mock responses for different models
const mockResponses = {
  "gpt-4": [
    "I'm GPT-4, OpenAI's most advanced language model. I can help you with a wide range of tasks including writing, analysis, coding, math, and creative projects. What would you like to work on today?",
    "That's an interesting question! Let me provide you with a comprehensive answer based on my training data.",
    "I'd be happy to help you with that. Here's what I think about your request...",
    "Great question! I can analyze this from multiple perspectives and provide you with detailed insights.",
    "I understand what you're looking for. Let me break this down step by step for you.",
  ],
  "gpt-3.5-turbo": [
    "Hello! I'm GPT-3.5 Turbo, a fast and efficient AI assistant. I'm here to help you with various tasks quickly and accurately.",
    "Great question! I can provide you with a quick and helpful response.",
    "I'm designed to be both fast and helpful. Let me assist you with that.",
    "That's a good point! Here's my take on it.",
    "I can help you with that efficiently and accurately.",
  ],
  "claude-3-sonnet": [
    "Hi there! I'm Claude 3 Sonnet from Anthropic. I'm designed to be helpful, harmless, and honest in all my interactions.",
    "That's a thoughtful question. I'll do my best to provide you with a balanced and informative response.",
    "I appreciate you asking! Let me share my thoughts on this topic.",
    "I'm happy to help with that. I'll approach this carefully and thoughtfully.",
    "That's an interesting perspective. Let me consider this from multiple angles.",
  ],
  "claude-3-haiku": [
    "Hello! I'm Claude 3 Haiku, optimized for quick and concise responses while maintaining quality.",
    "Quick response: I can help with that efficiently!",
    "Fast and helpful - that's my specialty!",
    "Concise answer coming right up!",
    "I'll keep this brief but informative.",
  ],
  "gemini-pro": [
    "Greetings! I'm Gemini Pro from Google. I excel at multimodal understanding and can help with text, images, and complex reasoning tasks.",
    "Interesting! I can process and understand multiple types of information to give you comprehensive answers.",
    "As a multimodal AI, I can help you with various types of content and analysis.",
    "I'm designed to handle complex reasoning and multimodal tasks. How can I assist you?",
    "That's a great question! I can provide detailed analysis and insights.",
  ],
  "llama-2-70b": [
    "Hey! I'm Llama 2 70B from Meta. I'm an open-source language model that's transparent and community-driven.",
    "Thanks for your question! As an open-source model, I aim to be transparent and helpful.",
    "I'm here to assist you with the power of open-source AI!",
    "Open-source AI at your service! How can I help you today?",
    "As a community-driven model, I'm here to provide helpful and transparent responses.",
  ],
};

// Methods
const selectModel = (model) => {
  selectedModelId.value = model.id;
  currentModel.value = model;
  isModelDropdownOpen.value = false;
};

const handleSubmit = () => {
  if (!inputMessage.value.trim() || isTyping.value) return;
  sendMessage(inputMessage.value.trim());
};

const sendMessage = async (content) => {
  try {
    const userMessage = {
      id: Date.now(),
      role: "user",
      content: content,
      timestamp: new Date(),
    };

    messages.value.push(userMessage);
    inputMessage.value = "";
    isTyping.value = true;

    const res = await axios.post(`${apiUrl}/chat`, {
      message: content,
      modelId: currentModel.value.id,
    });

    messages.value.push(res.data.chatHistory[res.data.chatHistory.length - 1]);
    isTyping.value = false;

    console.log(res);
  } catch (e) {
    alert("Unable to submit request");
  }

  return;
};

const copyMessage = async (content, messageId) => {
  try {
    await navigator.clipboard.writeText(content);
    copiedMessageId.value = messageId;
    setTimeout(() => {
      copiedMessageId.value = null;
    }, 2000);
  } catch (err) {
    console.error("Failed to copy text:", err);
  }
};

const regenerateResponse = (messageId) => {
  const messageIndex = messages.value.findIndex((m) => m.id === messageId);
  if (messageIndex > 0) {
    const previousUserMessage = messages.value[messageIndex - 1];
    messages.value.splice(messageIndex); // Remove the AI message and any after it
    sendMessage(previousUserMessage.content);
  }
};

const clearChat = () => {
  messages.value = [];
};

const autoResize = (event) => {
  const textarea = event.target;
  textarea.style.height = "auto";
  textarea.style.height = Math.min(textarea.scrollHeight, 120) + "px";
};

const scrollToBottom = () => {
  if (messagesContainer.value) {
    messagesContainer.value.scrollTop = messagesContainer.value.scrollHeight;
  }
};

const formatTime = (date) => {
  return typeof date == "string"
    ? date
    : date.toLocaleTimeString([], { hour: "2-digit", minute: "2-digit" });
};

const fetchSupportedModels = async () => {
  try {
    const res = await axios.get(`${apiUrl}/supported-models`);
    models.value = res.data?.models;
    currentModel.value = res.data?.selectedModel;
    selectedModelId.value = currentModel.id;

    console.log(currentModel.value);
  } catch (err) {
    console.error("Unable to fetch supported models", err);
  }
};

const chatHistory = async () => {
  try {
    const res = await axios.get(`${apiUrl}/chat-history`);
    messages.value = res.data.chats;
  } catch (err) {
    console.error("Unable to fetch supported models", err);
  }
};

// Lifecycle
onMounted(async () => {
  await chatHistory();
  await fetchSupportedModels();

  if (messageInput.value) {
    messageInput.value.focus();
  }
});
</script>
