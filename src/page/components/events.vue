<template>
  <div class="container">
    <div class="section">
      <h2>Authenticated</h2>
      <div class="space-between">
        <span>{{ neko.state.authenticated }}</span>
      </div>
      <div class="space-between">
        <strong>ID:</strong>
        <span>{{ neko.state.session_id }}</span>
      </div>
      <div class="space-between">
        <span>Total {{ Object.values(neko.state.sessions).length }} sessions.</span>
      </div>
    </div>

    <div class="section">
      <h2>Connection</h2>
      <div class="space-between">
        <strong>URL:</strong>
        <span>{{ neko.state.connection.url }}</span>
      </div>
      <div class="space-between">
        <strong>Token:</strong>
        <span>{{ neko.state.connection.token ? 'yes' : 'no' }}</span>
      </div>
      <div class="space-between">
        <strong>Status:</strong>
        <span>{{ neko.state.connection.status }}</span>
      </div>
      <div class="space-between">
        <strong>Websocket Connected:</strong>
        <span>{{ neko.state.connection.websocket.connected }}</span>
      </div>
      <details>
        <summary>Websocket Config</summary>
        <div class="space-between">
          <strong>Max Reconnects:</strong>
          <span>{{ neko.state.connection.websocket.config.max_reconnects }}</span>
        </div>
        <div class="space-between">
          <strong>Timeout (ms):</strong>
          <span>{{ neko.state.connection.websocket.config.timeout_ms }}</span>
        </div>
        <div class="space-between">
          <strong>Backoff (ms):</strong>
          <span>{{ neko.state.connection.websocket.config.backoff_ms }}</span>
        </div>
      </details>
      <div class="space-between">
        <strong>WebRTC Connected:</strong>
        <span>{{ neko.state.connection.webrtc.connected }}</span>
      </div>
      <div class="space-between">
        <strong>WebRTC Stable:</strong>
        <span>{{ neko.state.connection.webrtc.stable }}</span>
      </div>
      <details>
        <summary>WebRTC Config</summary>
        <div class="space-between">
          <strong>Max Reconnects:</strong>
          <span>{{ neko.state.connection.webrtc.config.max_reconnects }}</span>
        </div>
        <div class="space-between">
          <strong>Timeout (ms):</strong>
          <span>{{ neko.state.connection.webrtc.config.timeout_ms }}</span>
        </div>
        <div class="space-between">
          <strong>Backoff (ms):</strong>
          <span>{{ neko.state.connection.webrtc.config.backoff_ms }}</span>
        </div>
      </details>
    </div>

    <div class="section">
      <h2>Settings</h2>
        <button @click="updateSettings({ private_mode: !neko.state.settings.private_mode })" class="toggle-button">
          {{ neko.state.settings.private_mode ? 'Disable Private Mode' : 'Enable Private Mode' }}
        </button>
      <div class="space-between">
        <button @click="updateSettings({ locked_logins: !neko.state.settings.locked_logins })" class="toggle-button">
          {{ neko.state.settings.locked_logins ? 'Disable Locked Logins' : 'Enable Locked Logins' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="updateSettings({ locked_controls: !neko.state.settings.locked_controls })" class="toggle-button">
          {{ neko.state.settings.locked_controls ? 'Disable Locked Controls' : 'Enable Locked Controls' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="updateSettings({ control_protection: !neko.state.settings.control_protection })" class="toggle-button">
          {{ neko.state.settings.control_protection ? 'Disable Control Protection' : 'Enable Control Protection' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="updateSettings({ implicit_hosting: !neko.state.settings.implicit_hosting })" class="toggle-button">
          {{ neko.state.settings.implicit_hosting ? 'Disable Implicit Hosting' : 'Enable Implicit Hosting' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="updateSettings({ inactive_cursors: !neko.state.settings.inactive_cursors })" class="toggle-button">
          {{ neko.state.settings.inactive_cursors ? 'Disable Inactive Cursors' : 'Enable Inactive Cursors' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="updateSettings({ merciful_reconnect: !neko.state.settings.merciful_reconnect })" class="toggle-button">
          {{ neko.state.settings.merciful_reconnect ? 'Disable Merciful Reconnect' : 'Enable Merciful Reconnect' }}
        </button>
      </div>
    </div>

    <div class="section">
      <h2>WebRTC Video</h2>
      <button @click="neko.setWebRTCAudio({ disabled: !neko.state.connection.webrtc.audio.disabled })" class="toggle-button">
          {{ neko.state.connection.webrtc.audio.disabled ? 'Enable WebRTC Audio' : 'Disable WebRTC Audio' }}
        </button>
      <div class="space-between">
        <button @click="neko.setWebRTCVideo({ disabled: !neko.state.connection.webrtc.video.disabled })" class="toggle-button">
          {{ neko.state.connection.webrtc.video.disabled ? 'Enable Video' : 'Disable Video' }}
        </button>
      </div>
      <div class="space-between">
        <button @click="neko.setWebRTCVideo({ auto: !neko.state.connection.webrtc.video.auto })" class="toggle-button">
          {{ neko.state.connection.webrtc.video.auto ? 'Disable Auto Video' : 'Enable Auto Video' }}
        </button>
      </div>
      <div class="space-between">
        <strong>Bitrate:</strong>
        <span>{{ Math.floor(neko.state.connection.webrtc.stats.bitrate / 1024 / 8) }} KB/s</span>
      </div>
      <div class="space-between">
        <strong>Latency:</strong>
        <span>{{ neko.state.connection.webrtc.stats.latency }}ms</span>
      </div>
      <div class="space-between">
        <strong>Packet Loss:</strong>
        <span>{{ Math.floor(neko.state.connection.webrtc.stats.packetLoss) }}%</span>
      </div>
      <div></div>
      <div class="space-between">
        <strong>Codec:</strong>
        <select
          :value="neko.state.connection.webrtc.video.id"
          @input="neko.setWebRTCVideo({ selector: { id: ($event.target as HTMLSelectElement).value || '' } })"
        >
          <option v-for="video in neko.state.connection.webrtc.videos" :key="video" :value="video">
            {{ video }}
          </option>
        </select>
      </div>
      <div>Total {{ neko.state.connection.webrtc.videos.length }} videos.</div>
    </div>

    <div class="section">
      <h2>Connection Type</h2>
      <div class="space-between">
        <span>{{ neko.state.connection.type }}</span>
      </div>
      <h2>Screencast</h2>
      <div class="space-between">
        <span>{{ neko.state.connection.screencast }}</span>
      </div>
    </div>

    <div class="section">
      <h2>Video</h2>
      <div class="space-between">
        <strong>Playable:</strong>
        <span>{{ neko.state.video.playable }}</span>
      </div>
      <div class="space-between">
        <button @click="neko.state.video.playing ? neko.pause() : neko.play()" class="toggle-button">
          {{ neko.state.video.playing ? 'Pause' : 'Play' }}
        </button>
      </div>
      <div>
        <strong>Volume:</strong>
        <input
          type="range"
          min="0"
          max="1"
          :value="neko.state.video.volume"
          @input="neko.setVolume(Number(($event.target as HTMLInputElement).value))"
          step="0.01"
        />
      </div>
      <div class="space-between">
        <button @click="neko.state.video.muted ? neko.unmute() : neko.mute()" class="toggle-button">
          {{ neko.state.video.muted ? 'Unmute' : 'Mute' }}
        </button>
      </div>
    </div>

    <div class="section">
      <h2>Control</h2>
      <h3>Scroll</h3>
      <div class="space-between">
        <button @click="neko.setScrollInverse(!neko.state.control.scroll.inverse)" class="toggle-button">
          {{ neko.state.control.scroll.inverse ? 'Disable Inverse Scroll' : 'Enable Inverse Scroll' }}
        </button>
      </div>
      <div>
        <strong>Sensitivity:</strong>
        <input
          type="range"
          min="-5"
          max="5"
          :value="neko.state.control.scroll.sensitivity"
          @input="neko.setScrollSensitivity(Number(($event.target as HTMLInputElement).value))"
          step="1"
        />
      </div>

      <h3>Clipboard</h3>
      <textarea
        :readonly="!neko.controlling"
        :value="neko.state.control.clipboard ? neko.state.control.clipboard.text : ''"
        @input="clipboardText = ($event.target as HTMLTextAreaElement).value"
      ></textarea>
      <button :disabled="!neko.controlling" @click="neko.room.clipboardSetText({ text: clipboardText })">
        Send Clipboard
      </button>

      <h3>Keyboard</h3>
      <div class="space-between">
        <span>{{ neko.state.control.keyboard.layout + (neko.state.control.keyboard.variant ? ' (' + neko.state.control.keyboard.variant + ')' : '') }}</span>
      </div>
      <input
        type="text"
        placeholder="Layout"
        :value="neko.state.control.keyboard.layout"
        @input="neko.setKeyboard(($event.target as HTMLInputElement).value, neko.state.control.keyboard.variant)"
      />
      <input
        type="text"
        placeholder="Variant"
        :value="neko.state.control.keyboard.variant"
        @input="neko.setKeyboard(neko.state.control.keyboard.layout, ($event.target as HTMLInputElement).value)"
      />

      <h3>Touch</h3>
      <div class="space-between">
        <button @click="neko.setTouchEnabled(!neko.state.control.touch.enabled)" class="toggle-button">
          {{ neko.state.control.touch.enabled ? 'Disable Touch' : 'Enable Touch' }}
        </button>
      </div>
      <div class="space-between">
        <strong>Supported:</strong>
        <span>{{ neko.state.control.touch.supported }}</span>
      </div>

      <h3>Host</h3>
      <div class="space-between">
        <strong>ID:</strong>
        <span>{{ neko.state.control.host_id }}</span>
      </div>
      <div>
        <button @click="neko.controlling ? neko.room.controlRelease() : neko.room.controlRequest()" class="toggle-button">
          {{ neko.controlling ? 'Release Control' : 'Request Control' }}
        </button>
      </div>
    </div>

    <div class="section">
      <h2>Screen</h2>
      <div class="space-between">
        <strong>Size:</strong>
        <span>{{ neko.state.screen.size.width }}x{{ neko.state.screen.size.height }}@{{ neko.state.screen.size.rate }}</span>
      </div>

      <div v-if="neko.is_admin">
        <h3>Configurations</h3>
        <div>Total {{ neko.state.screen.configurations.length }} configurations.</div>
        <select id="screen-configuration" v-model="screenConfiguration">
          <option
            v-for="{ width, height, rate } in neko.state.screen.configurations"
            :key="String(width) + 'x' + String(height) + '@' + String(rate)"
            :value="String(width) + 'x' + String(height) + '@' + String(rate)"
          >
            {{ String(width) + 'x' + String(height) + '@' + String(rate) }}
          </option>
        </select>
        <button class="small-buttons" @click="setScreenConfiguration">Set Configuration</button>

        <h3></h3>
        <div class="space-between">
          <button @click="neko.state.screen.sync.enabled = !neko.state.screen.sync.enabled" class="toggle-button">
            {{ neko.state.screen.sync.enabled ? 'Disable Screen Sync' : 'Enable Screen Sync' }}
          </button>
        </div>
        <div>
          <strong>Multiplier:</strong>
          <input
            type="range"
            min="0"
            max="10"
            :value="neko.state.screen.sync.multiplier"
            @input="neko.state.screen.sync.multiplier = Number(($event.target as HTMLInputElement).value)"
            step="0.1"
          />
        </div>
        <div>
          <strong>Rate:</strong>
          <input
            type="range"
            min="5"
            max="60"
            :value="neko.state.screen.sync.rate"
            @input="neko.state.screen.sync.rate = Number(($event.target as HTMLInputElement).value)"
            step="5"
          />
        </div>
      </div>

      <div v-else class="space-between">
        <span>Session is not admin.</span>
      </div>
    </div>


    <div class="section">
      <h2>Cursors</h2>
      <div class="space-between">
        <span>{{ neko.state.cursors }}</span>
      </div>
    </div>

    <div class="section">
      <h2>Mobile Keyboard Open</h2>
        <button @click="neko.mobileKeyboardToggle" class="toggle-button">
          {{ neko.state.mobile_keyboard_open ? 'Close Keyboard' : 'Open Keyboard' }}
        </button>
    </div>

    <div class="section">
      <h2>Control Actions</h2>
      <button class="control-buttons" title="cut" @click="neko.control.cut()"><i class="fas fa-cut control-buttons" /></button>
      <button class="control-buttons" title="copy" @click="neko.control.copy()"><i class="fas fa-copy control-buttons" /></button>
      <button class="control-buttons" title="paste" @click="neko.control.paste()">
        <i class="fas fa-paste control-buttons" />
      </button>
      <button class="control-buttons" title="select all" @click="neko.control.selectAll()">
        <i class="fas fa-i-cursor control-buttons" />
      </button>
    </div>

    <div class="section">
      <h2>Control Keypress</h2>
      <button v-for="l in letters" :key="l" @click="neko.control.keyPress(l)" class="small-buttons">
        {{ String.fromCharCode(l) }}
      </button>
      <div>
      <div></div>
      <button title="shift" @click="shift = !shift" class="small-buttons">
        <i v-if="shift" class="fas fa-caret-square-up small-buttons" />
        <i v-else class="far fa-caret-square-up small-buttons" />
      </button>
      <button @click="neko.control.keyPress(' '.charCodeAt(0))" class="small-buttons">
        space
      </button>
    </div>
    </div>


    <div class="section">
      <h2>Chaos Monkey</h2>
      <button @click="cursorMovingToggle">cursor moving</button>
      <button @click="screenChangingToggle">screen cfg changing</button>
    </div>
    <div class="section">
    <h2>Color Picker</h2>
    <NekoColor @colorChange="neko.control.buttonPress(1, $event)" class="left-box"/>
    </div>
  </div>
</template>

<style lang="scss">
@import '@/page/assets/styles/main.scss';
$section-border: 1px solid #ccc;
$section-padding: 10px;
$section-radius: 5px;
.left-box {
  width: 75%;
  margin-left: 0;
  margin-right: 0;
}
.toggle-button {
  width: 75%;
  margin-left: 12.5%;
  font-size: 0.8em;
  margin-top: 5px;
  align-self: center;
  justify-self: center;
}
.small-buttons {
  width: fit-content;
  font-size: 1.2em;
  margin: 0px;
}
.control-buttons {
  width: fit-content;
  margin-left: 12.5%;
  font-size: 1em;
  align-self: center;
  justify-self: center;
}
button {
  color: white;
margin-top: 5px;
width: 75%;
margin-left: 12.5%;
align-self: center;
justify-self: center;
border: 0.5px solid white;
background-color: #9d9d9d;
accent-color: $style-primary;
cursor: pointer;

&:hover {
  color: black;
  background-color: #ffffff;
}

&.toggle-button {
  width: 75%;
  font-size: 0.8em;
  margin-top: 5px;
  margin-left: 12.5%;
  align-self: center;
  justify-self: center;
}
&.small-buttons {
  font-size: 0.8emem;
  // padding: 5px;
  width: fit-content;
  // margin: 5px;
}
&.control-buttons {
  font-size: 1em;
  margin-left: 12.5%;
  width: fit-content;
  align-self: center;
  justify-self: center;
}
}
.container {
  width: 100%;
  max-width: 100%;
  height: 100vh;
  overflow-y: auto;
  overflow-x: hidden;
  padding: 10px;
  box-sizing: border-box;
  align-items: center;

  .section {
    border: $section-border;
    padding: $section-padding;
    border-radius: $section-radius;
    align-items: center;
    margin-bottom: 20px;

    h2 {
      margin-top: 0;
      text-align: center;
      align-self: center;
    }

    .space-between {
      display: flex;
      justify-content: space-between;
      align-items: center;
      // width: 100%;
    }

    strong {
      font-weight: bold;
    }

    button {
      color: white;
      margin-top: 5px;
      width: 75%;
      margin-left: 12.5%;
      align-self: center;
      justify-self: center;
      border: 0.5px solid white;
      background-color: #242424;
      accent-color: $style-primary;
      cursor: pointer;

      &:hover {
        color: black;
        background-color: #ddd;
      }

      &.toggle-button {
        width: 75%;
        font-size: 0.8em;
        margin-top: 5px;
        margin-left: 12.5%;
        align-self: center;
        justify-self: center;
      }
      &.small-buttons {
        font-size: 0.8emem;
        margin: 0px;
        // padding: 5px;
        width: fit-content;
        // margin: 5px;
      }
    }

    input[type='range'] {
      accent-color: $style-primary;
      width: 100%;
      background-color: #242424;
      ::-webkit-slider-runnable-track {
        background: #242424;
        border: 0.2px solid white;
        border-radius: 1.3px;
        width: 100%;
        height: 1.3px;
      }
      ::-moz-range-track {
        background: #242424;
        border: 0.2px solid white;
        border-radius: 1.3px;
        width: 100%;
        height: 1.3px;
      }
    }


    textarea {
      width: 100%;
      height: 80px;
      resize: vertical;
      background-color: #242424;
      color: white;
    }

    /* Center buttons within sections */
    h2 + .space-between {
      display: flex;
      justify-content: center;
      flex-wrap: wrap;
      gap: 10px;

      button {
        flex: 1 0 auto;
        min-width: 60px;
        width: 50%;
      }

    }
  }
}
</style>


<script lang="ts" setup>
import { ref, watch, computed } from 'vue'
import type Neko from '@/component/main.vue'
import NekoColor from './color.vue'

const props = defineProps<{
  neko: typeof Neko
}>()

const clipboardText = ref('')
const bitrate = ref<number | null>(null)

watch(() => props.neko.state.connection.webrtc.bitrate, (val) => {
  bitrate.value = val
})

const shift = ref(false)
const letters = computed(() => {
  let letters = [] as number[]
  for (let i = (shift.value ? 'A' : 'a').charCodeAt(0); i <= (shift.value ? 'Z' : 'z').charCodeAt(0); i++) {
    letters.push(i)
  }
  return letters
})

// fast sceen changing test
let screen_interval: number | null = null
function screenChangingToggle() {
  if (screen_interval === null) {
    let sizes = props.neko.state.screen.configurations
    let len = sizes.length

    screen_interval = window.setInterval(() => {
      let { width, height, rate } = sizes[Math.floor(Math.random() * len)]

      props.neko.setScreenSize(width, height, rate)
    }, 10)
  } else {
    window.clearInterval(screen_interval)
    screen_interval = null
  }
}

const screenConfiguration = ref('')
function setScreenConfiguration() {
  let [width, height, rate] = screenConfiguration.value.split(/[@x]/)
  props.neko.setScreenSize(parseInt(width), parseInt(height), parseInt(rate))
}

watch(() => props.neko.state.screen.size, (val) => {
  screenConfiguration.value = `${val.width}x${val.height}@${val.rate}`
})

// fast cursor moving test
let cursor_interval: number | null = null
function cursorMovingToggle() {
  if (cursor_interval === null) {
    let len = props.neko.state.screen.size.width

    cursor_interval = window.setInterval(() => {
      let x = Math.floor(Math.random() * len)
      let y = Math.floor(Math.random() * len)

      props.neko.control.move({ x, y })
    }, 10)
  } else {
    window.clearInterval(cursor_interval)
    cursor_interval = null
  }
}

async function updateSettings(settings: any) {
  try {
    await props.neko.room.settingsSet(settings)
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}
</script>
