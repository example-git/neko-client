<template>
  <div class="chat">
    <div class="clear-button" @click="clearChat">
      <i class="fas fa-rotate" v-b-tooltip.hover title="Clear Chat"
      :class="{ 'fa-spin': isHovering }"
      @mouseover="isHovering = true" 
      @mouseout="isHovering = false"></i> 
    </div>
    <ul class="chat-history" ref="history">
      <li 
        v-for="(message, index) in messagesWithColors" 
        :key="index"
        class="message" 
        :class="{ alternate: index % 2 === 1, lastMessage: index === messagesWithColors.length - 1 }"
      >
        <div class="user-pfp">
          <!-- Placeholder -->
          <img src="../../page/assets/pfp-placeholder.png" alt="User Profile Picture">
        </div>
        <div class="content">
          <div class="content-head">
            <span :style="{ color: message.color }" class="session">{{ session(message.id) }}</span>
            <span class="timestamp">{{ timestamp(message.created) }}</span>
          </div>
          <p>{{ message.content }}</p>
        </div>
      </li>
    </ul>
    <div class="chat-send">
      <div class="text-container">
        <textarea 
          ref="input" 
          placeholder="Send a message" 
          @keydown="onKeyDown" 
          v-model="content" 
        />
      </div>
    </div>
  </div>
  <audio ref="audioElement" src="../../page/assets/chat.mp3" preload="auto"></audio> 
</template>
<style lang="scss" scoped>
  @import '@/page/assets/styles/main.scss';

  .chat {
    position: relative;
    flex: 1;
    flex-direction: column;
    display: flex;
    max-height: 100%;
    max-width: 100%;
    overflow-x: hidden;

    .clear-button {
      position: absolute; 
      top: 10px;
      right: 10px;  
      cursor: pointer;
      color: $text-muted; 
      &:hover {
        color: $text-normal; 
      }
      i {
        font-size: 2.5em; // Doubles the default font size
        transition: transform 0.2s ease; // Add a transition for a smooth spin effect
      }

      &:hover i { 
        transform: rotate(360deg); // Rotate icon on hover
      }
    }
    .chat-history {
      flex: 1;
      overflow-y: scroll;
      overflow-x: hidden;
      max-width: 100%;
      scrollbar-width: thin;
      scrollbar-color: $background-tertiary transparent;

      &::-webkit-scrollbar {
        width: 8px;
      }

      &::-webkit-scrollbar-track {
        background-color: transparent;
      }

      &::-webkit-scrollbar-thumb {
        background-color: $background-tertiary;
        border: 2px solid $background-primary;
        border-radius: 4px;
      }

      &::-webkit-scrollbar-thumb:hover {
        background-color: $background-floating;
      }

      .message {
        flex: 1;
        border-top: 1px solid var(--border-color);
        padding: 10px 10px 10px 10px;
        display: flex;
        flex-direction: row; // Change from column to row
        align-items: flex-start; // Align items to the start of the cross axis
        flex-wrap: nowrap;
        overflow: hidden;
        user-select: text;
        word-wrap: break-word;
        font-size: 16px;
        font: 'Calibri', sans-serif;

        &.alternate {
          background-color: darken($background-primary, 5%); // Adjust darkening percentage as needed
          font: 'Lucida Sans Unicode';
        }
      }

      .user-pfp {
        width: 30px;
        height: 30px;
        margin-right: 10px;
        margin-bottom: 10px;
        flex-shrink: 0; // Prevent the profile picture from shrinking


        img {
          width: 100%;
          height: 100%;
          border-radius: 50%; // Makes the pfp circular
        }
      }

      .content {
        flex: 1; // This will allow the message content to expand
        display: flex;
        flex-direction: column; // Ensures the heading and message stay in a column
      }


      .content-head {
        cursor: default;
        width: 100%;
        margin-bottom: 3px;
        display: block;

        .session {
          display: inline-block;
          color: $text-normal;
          font-weight: bold;
        }

        .timestamp {
          color: $text-muted;
          font-size: 0.7rem;
          margin-left: 0.3rem;
          line-height: 12px;
          &::first-letter {
            text-transform: uppercase;
          }
        }
      }
    }

    .chat-send {
      flex-shrink: 0;
      height: 80px;
      max-height: 80px;
      padding: 10px 10px 10px 10px;
      flex-direction: column;
      display: flex;

      .text-container {
        flex: 1;
        width: 100%;
        height: 100%;
        background-color: rgba($color: #fff, $alpha: 0.05);
        border-radius: 5px;
        position: relative;
        display: flex;

        .emoji-menu {
          width: 20px;
          height: 20px;
          font-size: 20px;
          margin: 8px 5px 0 0;
          cursor: pointer;
        }

        textarea {
          flex: 1;
          font-family: $text-family;
          border: none;
          caret-color: $text-normal;
          color: $text-normal;
          resize: none;
          margin: 5px;
          background-color: transparent;
          scrollbar-width: thin;
          scrollbar-color: $background-tertiary transparent;

          &::placeholder {
            color: $text-muted;
          }

          &::-webkit-scrollbar {
            width: 4px;
          }

          &::-webkit-scrollbar-track {
            background-color: transparent;
          }

          &::-webkit-scrollbar-thumb {
            background-color: $background-tertiary;
            border-radius: 4px;
          }

          &::-webkit-scrollbar-thumb:hover {
            background-color: $background-floating;
          }

          &::selection {
            background: $text-link;
          }
        }
      }
    }
  }
</style>

<script lang="ts" setup>
import { ref, watch, onMounted, computed, onUnmounted, nextTick } from 'vue';
import type Neko from '@/component/main.vue';


const length = 512; // max length of message

const history = ref<HTMLUListElement | null>(null);
// Correct way to create a ref for a dynamic element:
const lastMessageRef = ref<HTMLLIElement | null>(null); 
const MAX_CACHED_MESSAGES = 100; 
const props = defineProps<{
  neko: typeof Neko;
}>();

const emit = defineEmits(['send_message']);

interface Message {
  id: string;
  created: Date;
  content: string;
  color?: string; // Add a color property to Message
}



const audioElement = ref() as { value: HTMLAudioElement | null };
const messages = ref<Message[]>([]);
const content = ref('');
// Correctly type the computed property
const allMessages = computed<Message[]>(() => [...messages.value]);
// Correctly initialize cachedMessages 
const cachedMessages = ref<Message[]>([]); 
const isHovering = ref(false); 


onMounted(() => {
  const storedMessages = localStorage.getItem('chatHistory');
  if (storedMessages) {
    cachedMessages.value = JSON.parse(storedMessages).slice(-MAX_CACHED_MESSAGES); 
  } else {
    cachedMessages.value = [];
  }
  messages.value = [...cachedMessages.value];

  // After setting messages, scroll to the bottom
  setTimeout(() => {
    scrollToBottom()
  }, 0);
});

const scrollToBottom = () => {
  if (history.value) {
    history.value.scrollTop = history.value.scrollHeight;
  }
};



setTimeout(() => {
  if (history.value) {
    history.value.scrollIntoView({ behavior: 'smooth' })
  }
}, 0);

// Save messages to local storage on component unmount
onUnmounted(() => {
  localStorage.setItem('chatHistory', JSON.stringify(messages.value));
});

// color values for username mapping
const pastelColors = [
  '#E6E6FA', '#B0E0E6', '#FFC0CB', '#FFE4E1', '#D3FFFA', 
  '#FFFFE0', '#FAFAD2', '#90EE90', '#FFDAB9', '#FFB6C1',  // Light Pink
  '#A9A9A9', '#F08080', '#FFA07A', '#FA8072', '#E9967A',  // More vibrant pastels
  '#F5DEB3', '#DEB887', '#D2B48C', '#BC8F8F', '#F4A460',
  '#DAA520' // Even more diverse pastel shades!
]; 

const userColorMap = new Map<string, string>(); // Store user-color mappings

function hashCode(str: string): number {
  let hash = 0;
  for (let i = 0; i < str.length; i++) {
    hash = str.charCodeAt(i) + ((hash << 5) - hash);
  }
  return hash;
}

function getUserColor(userId: string): string {
  const index = Math.abs(hashCode(userId)) % pastelColors.length; 
  return pastelColors[index];
}

const messagesWithColors = computed(() => {
  return messages.value.map(message => ({
    ...message,
    color: getUserColor(message.id)
  }));
});

function intToHexColor(i: number): string {
  let c = (i & 0x00FFFFFF).toString(16).toUpperCase();
  c = '80' + c.substring(2); // Ensure the first two hex digits are '80' or higher
  return '#' + '00000'.substring(0, 6 - c.length) + c;
}

function onMessageReceived(message: Message) {
  messages.value = [...messages.value, message];
}

function timestamp(date: Date | string) {
  date = new Date(date)

  return (
    date.getFullYear() +
    '-' +
    String(date.getMonth() + 1).padStart(2, '0') +
    '-' +
    String(date.getDate()).padStart(2, '0') +
    ' ' +
    String(date.getHours()).padStart(2, '0') +
    ':' +
    String(date.getMinutes()).padStart(2, '0') +
    ':' +
    String(date.getSeconds()).padStart(2, '0')
  )
}

function session(id: string) {
  let session = props.neko.state.sessions[id]
  return session ? session.profile.name : id
}

function save_chat() {
  localStorage.setItem('chatHistory', JSON.stringify(messages.value));

}

function onHistroyChange() {
  setTimeout(() => {
    if (history.value) {
      // history.value.scrollTop = history.value.scrollHeight;
      history.value.scrollIntoView({ behavior: 'smooth' })
    }
  }, 0);
}

watch(allMessages, onHistroyChange, { deep: true });
watch(allMessages, save_chat)

function onKeyDown(event: KeyboardEvent) {
  if (content.value.length > length) {
    content.value = content.value.substring(0, length)
  }

  if (content.value.length == length) {
    if (
      [8, 16, 17, 18, 20, 33, 34, 35, 36, 37, 38, 39, 40, 45, 46, 91, 93, 144].includes(event.keyCode) ||
      (event.ctrlKey && [67, 65, 88].includes(event.keyCode))
    ) {
      return
    }
    event.preventDefault()
    return
  }

  if (event.keyCode !== 13 || event.shiftKey) {
    return
  }

  if (content.value === '') {
    event.preventDefault()
    return
  }

  emit('send_message', content.value)

  let message = {
    id: props.neko.state.session_id,
    created: new Date(),
    content: content.value,
  }
 
  props.neko.sendBroadcast('chat', message)
  onMessageReceived(message)
  content.value = ''
  event.preventDefault()
}

function playNotificationSound() {
  if (audioElement.value) {
    audioElement.value.play().catch((error) => {
      console.error('Error playing notification sound:', error);
      alert('Sorry, we were unable to play the notification sound. Please check your browser settings.');
    });
  }
}
// Watch the 'messages' array for changes

function clearChat() {
  if (!confirm('Are you sure you want to clear the chat history? This action cannot be undone.')) return; 
  messages.value = [];
  cachedMessages.value = []; 
  localStorage.removeItem('chatHistory');
};


watch(messages, () => {
   setTimeout(() => {
     scrollToBottom()
   }, 0);
});


props.neko.events.on('receive.broadcast', (sender: string, subject: string, body: any) => {
  if (subject === 'chat') {
    const message = body as Message;
    onMessageReceived(message);
  }
  playNotificationSound();
});
</script>
