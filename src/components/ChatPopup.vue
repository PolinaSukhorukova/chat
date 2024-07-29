<template>
  <div>
    <div v-if="!isActive" class="toggle-button" @click="toggleChat">
      <svg class="svg-chat" style="enable-background:new 0 0 58 58;" version="1.1" viewBox="0 0 58 58" xml:space="preserve" xmlns="http://www.w3.org/2000/svg" xmlns:xlink="http://www.w3.org/1999/xlink"><g><path d="M25,9.586C11.193,9.586,0,19.621,0,32c0,4.562,1.524,8.803,4.135,12.343   C3.792,48.433,2.805,54.194,0,57c0,0,8.571-1.203,14.377-4.709c3.225,1.359,6.824,2.123,10.623,2.123c13.807,0,25-10.035,25-22.414   S38.807,9.586,25,9.586z" style="fill:#3BA58B;"/><path d="M58,23.414C58,11.035,46.807,1,33,1c-9.97,0-18.575,5.234-22.589,12.804   C14.518,11.153,19.553,9.586,25,9.586c13.807,0,25,10.035,25,22.414c0,4.735-1.642,9.124-4.437,12.743   C51.162,47.448,58,48.414,58,48.414c-2.805-2.805-3.792-8.566-4.135-12.657C56.476,32.217,58,27.976,58,23.414z" style="fill:#226BAC;"/><circle cx="12" cy="32" r="3" style="fill:#FFFFFF;"/><circle cx="25" cy="32" r="3" style="fill:#FFFFFF;"/><circle cx="38" cy="32" r="3" style="fill:#FFFFFF;"/></g><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/><g/></svg>
    </div>

    <div class="chat-popup" :class="{ 'is-active': isActive }">
      <div class="chat-header" @click="toggleChat">
        <span>{{ isActive ? 'Close' : 'Chat' }}</span>
      </div>
      <div class="chat-content" ref="chatContainer" v-if="isActive">
        <div v-for="(message, index) in messages" :key="index" :class="['chat-message', message.type]">
          <span>{{ message.text }}</span>
        </div>
        <div v-if="typing" class="typing-indicator">
          Typing<span class="dot"></span><span class="dot"></span><span class="dot"></span>
        </div>
        <div v-if="showOptions && !typing">
          <div v-for="(option, index) in options" :key="index" class="chat-option"
            @click="handleOptionClick(option.value)">
            <span>{{ option.text }}</span>
          </div>
        </div>
        <div v-if="showInput && !typing" class="chat-input-container">
          <input v-model="userInput" class="chat-input" @keyup.enter="submitQuestion" placeholder="Ваше питання" />
          <button @click="submitQuestion">Відправити</button>
        </div>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, nextTick, watch } from 'vue';
import axios from 'axios';

const isActive = ref(false);
const typing = ref(false);
const messages = ref([]);
const options = ref([]);
const showOptions = ref(false);
const showInput = ref(false);
const userInput = ref('');
const chatContainer = ref(null);

const toggleChat = () => {
  isActive.value = !isActive.value;
  if (isActive.value) {
    startChat();
  }
};

const startChat = async () => {
  typing.value = true;
  await delay(1000);
  typing.value = false;
  const data = await fetchData();
  addMessage({ text: data.greeting.greeting, type: 'received' });
  await delay(1000);
  addMessage({ text: data.greeting.message, type: 'received' });
  await delay(1000);
  options.value = data.options;
  showOptions.value = true;
};

const handleOptionClick = async (optionValue) => {
  addMessage({ text: options.value.find(option => option.value === optionValue).text, type: 'sent' });
  showOptions.value = false;
  typing.value = true;
  await delay(1000);
  typing.value = false;
  const response = await getResponse(optionValue);
  addMessage({ text: response, type: 'received' });
  if (optionValue === 'customQuestion') {
    showInput.value = true;
  } else {
    await delay(1000);
    const data = await fetchData();
    options.value = data.options;
    showOptions.value = true;
    nextTick(() => {
      const container = chatContainer.value;
      if (container) {
        container.scrollTop = container.scrollHeight;
      }
    });
  }
};

const submitQuestion = async () => {
  addMessage({ text: userInput.value, type: 'sent' });
  try {
    await axios.post('/api/questions', { question: userInput.value });
  } catch (error) {
    console.error(error);
  } finally {
    showInput.value = false;
  }
  userInput.value = '';
  showInput.value = false;
  typing.value = true;
  await delay(2000);
  typing.value = false;
  await delay(1000);
  addMessage({ text: 'Дякуємо за Ваше питання!', type: 'received' });
  await delay(1000);
  const data = await fetchData();
  options.value = data.options;
  showOptions.value = true;
  nextTick(() => {
    const container = chatContainer.value;
    if (container) {
      container.scrollTop = container.scrollHeight;
    }
  });
};

const delay = (ms) => new Promise((resolve) => setTimeout(resolve, ms));

const fetchData = async () => {
  const response = await axios.get('/chat/data.json');
  return response.data;
};

const getResponse = async (optionValue) => {
  const data = await fetchData();
  const responses = data.responses;
  const currentDate = new Date();
  const day = currentDate.toLocaleDateString("uk-UA", { weekday: "long" });
  const time = currentDate.toLocaleTimeString("uk-UA");
  const newYearDate = new Date(currentDate.getFullYear() + 1, 0, 1);
  const daysToNewYear = Math.floor((+newYearDate - +currentDate) / (1000 * 60 * 60 * 24));

  let responseText = responses[optionValue];
  if (optionValue === 'day') {
    responseText = responseText.replace('{{day}}', day);
  } else if (optionValue === 'time') {
    responseText = responseText.replace('{{time}}', time);
  } else if (optionValue === 'daysToNewYear') {
    responseText = responseText.replace('{{days}}', daysToNewYear);
  } else if (optionValue === 'customQuestion') {
    responseText = responseText.replace('{{question}}', userInput.value);
  }

  return responseText;
};

const addMessage = (message) => {
  messages.value.push(message);
  nextTick(() => {
    const container = chatContainer.value;
    if (container) {
      container.scrollTop = container.scrollHeight;
    }
  });
};

watch(messages, () => {
  nextTick(() => {
    const container = chatContainer.value;
    if (container) {
      container.scrollTop = container.scrollHeight;
    }
  });
});
</script>

<style scoped>
.svg-chat {
  width: 30px;
  height: 30px;
}
.chat-popup {
  position: fixed;
  bottom: 60px;
  right: 20px;
  width: 400px;
  height: 500px;
  box-sizing: border-box;
  background-color: white;
  border: 1px solid #ccc;
  box-shadow: 0px 0px 10px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s ease-in-out, opacity 0.3s ease-in-out;
  transform: translateY(100%);
  opacity: 0;
  display: flex;
  flex-direction: column;
}

.chat-popup.is-active {
  transform: translateY(0);
  opacity: 1;
}

.chat-header {
  background: #007bff;
  color: #fff;
  padding: 10px;
  cursor: pointer;
  box-sizing: border-box;
  text-align: center;
}

.chat-content {
  background: #f8f9fa;
  height: calc(100% - 40px);
  overflow-y: auto;
  padding: 10px;
  box-sizing: border-box;
  display: flex;
  flex-direction: column;
}

.chat-message {
  margin: 5px 0;
  padding: 10px;
  border-radius: 5px;
  max-width: 80%;
  word-break: break-word;
}

.chat-message.sent {
  background: #007bff;
  color: #fff;
  align-self: flex-end;
  text-align: right;
}

.chat-message.received {
  background: #e9ecef;
  color: #000;
  align-self: flex-start;
  text-align: left;
}

.chat-option {
  background: #e9ecef;
  padding: 10px;
  margin: 5px 0;
  border-radius: 5px;
  cursor: pointer;
}

.chat-option:hover {
  background: #ced4da;
}

.typing-indicator {
  display: flex;
  align-items: center;
}

.typing-indicator .dot {
  height: 8px;
  width: 8px;
  background-color: #007bff;
  border-radius: 50%;
  margin: 0 2px;
  animation: blink 1.4s infinite both;
}

.typing-indicator .dot:nth-child(2) {
  animation-delay: 0.2s;
}

.typing-indicator .dot:nth-child(3) {
  animation-delay: 0.4s;
}

@keyframes blink {
  0% {
    opacity: 0.2;
  }

  20% {
    opacity: 1;
  }

  100% {
    opacity: 0.2;
  }
}

.chat-input-container {
  display: flex;
  align-items: center;
  margin-top: auto;
  padding: 10px;
  background: #f8f9fa;
  border-top: 1px solid #ced4da;
}

.chat-input-container input {
  flex: 1;
  padding: 10px;
  border-radius: 5px;
  border: 1px solid #ced4da;
  background: #fff;
  font-size: 14px;
  margin-right: 10px;
  box-sizing: border-box;
}

.chat-input-container button {
  background: #007bff;
  color: #fff;
  border: none;
  padding: 10px 15px;
  border-radius: 5px;
  cursor: pointer;
}

.chat-input-container button:hover {
  background: #0056b3;
}

.toggle-button {
  position: fixed;
  bottom: 20px;
  right: 20px;
  cursor: pointer;
}
</style>