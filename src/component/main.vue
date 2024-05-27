<template>
    <div ref="fullscreenContainer" class="fullscreen-container"> 
      <div ref="component" class="neko-component">
      <div ref="container" class="neko-container">
      <button @click="enter" class="fullscreen-button small-buttons" v-if="!isFullscreen">
        <i class="fas fa-expand small-buttons"></i>
      </button>
      <button @click="exit" class="fullscreen-button small-buttons" v-if="isFullscreen">
        <i class="fas fa-compress small-buttons"></i>
      </button>
      <div class="screen-config-wrapper" v-if="is_admin">
          <div class="cog-icon-wrapper">
            <svg
              xmlns="http://www.w3.org/2000/svg"
              viewBox="0 0 448 512"
              class="cog-icon"
              @click="toggleDropdown()"
              @click.away="toggleDropdown()"
              @click.stop="toggleDropdown()"
              tabindex="0"
            >
              <path
                d="M0 96C0 78.3 14.3 64 32 64H416c17.7 0 32 14.3 32 32s-14.3 32-32 32H32C14.3 128 0 113.7 0 96zM0 256c0-17.7 14.3-32 32-32H416c17.7 0 32 14.3 32 32s-14.3 32-32 32H32c-17.7 0-32-14.3-32-32zM448 416c0 17.7-14.3 32-32 32H32c-17.7 0-32-14.3-32-32s14.3-32 32-32H416c17.7 0 32 14.3 32 32z"
              />
            </svg>
            <div v-show="showScreenConfigMenu" class="screen-config-menu">
              <div
                v-for="config in state.screen.configurations"
                :key="`${config.width}x${config.height}@${config.rate}`"
                @click="screenConfiguration = `${config.width}x${config.height}@${config.rate}`; toggleDropdown()"
                @click.stop="toggleDropdown()"
                @click.away="toggleDropdown()"
                class="screen-config-option"
              >
                {{ `${config.width}x${config.height}@${config.rate}` }}
              </div>
            </div>
          </div>
        </div>
      <video ref="video" playsinline></video>
      <Screencast
        v-show="screencast && screencastReady"
        :image="fallbackImage"
        :enabled="screencast || (!state.connection.webrtc.stable && state.connection.webrtc.connected)"
        :api="api.room"
        @imageReady="screencastReady = $event"
      />
      <Cursors
        v-if="state.settings.inactive_cursors && session && session.profile.can_see_inactive_cursors"
        :sessions="state.sessions"
        :sessionId="state.session_id || ''"
        :hostId="state.control.host_id"
        :screenSize="state.screen.size"
        :canvasSize="canvasSize"
        :cursors="state.cursors"
        :cursorDraw="inactiveCursorDrawFunction"
        :fps="fps"
      />
      <Overlay
        ref="overlay"
        v-show="!private_mode_enabled && state.connection.status != 'disconnected'"
        :style="{ pointerEvents: state.control.locked || (session && !session.profile.can_host) ? 'none' : 'auto' }"
        :control="control"
        :sessions="state.sessions"
        :hostId="state.control.host_id || ''"
        :webrtc="connection.webrtc"
        :scroll="state.control.scroll"
        :screenSize="state.screen.size"
        :canvasSize="canvasSize"
        :isControling="controlling"
        :cursorDraw="cursorDrawFunction"
        :implicitControl="!!(state.settings.implicit_hosting && session && session.profile.can_host)"
        :inactiveCursors="!!(state.settings.inactive_cursors && session && session.profile.sends_inactive_cursor)"
        :fps="fps"
        :hasMobileKeyboard="is_touch_device"
        @updateKeyboardModifiers="updateKeyboardModifiers($event)"
        @uploadDrop="uploadDrop($event)"
        @mobileKeyboardOpen="state.mobile_keyboard_open = $event"
      />
    </div>
    </div>
    </div>
</template>

<style lang="scss">
  .neko-component {
    width: 100%;
    height: 100%;
  }

  .fullscreen-container {
  width: 100%;
  height: 100%;
  position: relative; /* Ensure proper positioning */ 
  }
 
  .fullscreen-container:fullscreen { 
  width: 100%;
  height: 100%;
  position: relative; /* Ensure proper positioning */ 
  }


  .fullscreen-button {
    position: absolute;
    top: 10px;
    right: 10px;
    padding: 10px;
    background-color: transparent;
    border: none;
    color: white;
    font-size: 20px;
    cursor: pointer;
    z-index: 9999; // Choose a value higher than other elements
    .small-buttons {
      width: fit-content;
    }
  }

  .neko-container {
    position: relative;

    video,
    img {
      position: absolute;
      top: 0;
      bottom: 0;
      width: 100%;
      height: 100%;
      display: flex;
      background: transparent !important;

      &::-webkit-media-controls {
        display: none !important;
      }
    }
  }
.screen-config-wrapper {
  position: absolute;
  top: 10px;
  right: 50px;
  z-index: 9999;
}

.cog-icon-wrapper {
  cursor: pointer;
  position: relative;
}

.cog-icon {
  position: absolute;
  top: 1px;
  right: 5px;
  padding: 10px;
  width: 20px;
  height: 20px;
  z-index: 9999;
  cursor: pointer;
  fill: white;
}

.screen-config-menu {
  position: absolute;
  top: 30px; /* Adjust position relative to the cog icon */
  right: 0;
  border: 1px solid #cccccca2;
  border-radius: 5px;
  width: max-content; /* Adjust as needed */
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);

  .screen-config-option {
    text-align: center;
    padding: 2px;
    border:rgba(255, 255, 255, 0.2);
    cursor: pointer;
  }
  .screen-config-option:nth-child(even) {
    background-color: rgba(0, 0, 0, 0.688); /* Light gray for even rows */
    padding-right: 5px;
    padding-left: 5px;

  }
  .screen-config-option:nth-child(odd) {
    background-color: rgba(95, 95, 95, 0.688); /* Slightly darker gray for odd rows */
    padding-right: 5px;
    padding-left: 5px;
  }
  .screen-config-option:hover {
    color: black;
    background-color: #ffffffa3;
  }
}

.fade-enter-active, .fade-leave-active {
  transition: opacity 0.5s;
}

.fade-enter, .fade-leave-to /* .fade-leave-active in <2.1.8 */ {
  opacity: 0;
}

fullscreenContainer::-webkit-media-controls-fullscreen-button {
  position: absolute;
  top: 10px;
  right: 10px;
  padding: 10px;
  background-color: transparent;
  border: none;
  color: white;
  font-size: 20px;
  cursor: pointer;
  z-index: 9999; // Choose a value higher than other elements
}
</style>

<script lang="ts" setup>
import { ref, watch, computed, reactive, onMounted, onBeforeUnmount, defineEmits, onUnmounted } from 'vue'

//export * as ApiModels from './api/models'
//export * as StateModels from './types/state'
//export * as webrtcTypes from './types/webrtc'

import type { Configuration } from './api/configuration'
import type { AxiosInstance, AxiosProgressEvent } from 'axios'

import ResizeObserver from 'resize-observer-polyfill'

import { NekoApi } from './internal/api'
import type { SessionsApi, MembersApi, RoomApi } from './internal/api'
import { NekoConnection } from './internal/connection'
import { NekoMessages } from './internal/messages'
import { NekoControl } from './internal/control'
import { register as VideoRegister } from './internal/video'

import type { ReconnectorConfig } from './types/reconnector'
import * as EVENT from './types/events'
import type * as webrtcTypes from './types/webrtc'
import type NekoState from './types/state'
import type { CursorDrawFunction, InactiveCursorDrawFunction, Dimension } from './types/cursors'
import Overlay from './overlay.vue'
import Screencast from './screencast.vue'
import Cursors from './cursors.vue'
import { useFullscreen } from '@vueuse/core'

const SCREEN_SYNC_THROTTLE = 500 // wait 500ms before reacting to automatic screen size change

const isFirstMount = ref(true)
const component = ref<HTMLElement | null>(null)
const container = ref<HTMLElement | null>(null)
const fullscreenContainer = ref<HTMLElement | null>(null);
const video = ref<HTMLVideoElement | null>(null)
const overlay = ref<typeof Overlay | null>(null)
const showScreenConfigMenu = ref(false)
const volume = ref(100)

const { isFullscreen, enter, exit, toggle } = useFullscreen(fullscreenContainer)

// fallback image for webrtc reconnections:
// chrome shows black screen when closing webrtc connection, that's why
// we need to grab video image before closing connection ans show that
// while reconnecting, to not see black screen
const fallbackImage = ref('')

const api = new NekoApi()
const observer = new ResizeObserver(onResize)
const canvasSize = ref<Dimension>({ width: 0, height: 0 })
const cursorDrawFunction = ref<CursorDrawFunction | null>(null)
const inactiveCursorDrawFunction = ref<InactiveCursorDrawFunction | null>(null)

const props = defineProps({
  server: String,
  autologin: Boolean,
  autoconnect: Boolean,
  autoplay: Boolean,
  // fps for cursor rendering, 0 for no cap
  fps: { type: Number, default: 0 },
  // auto / touch / mouse
  inputMode: { type: String, default: 'auto' },
  showModal: Boolean,
  isFirstMount: Boolean,
})

const showModal = reactive<{ value: boolean }>({ value: props.showModal })


/////////////////////////////
// Public state
/////////////////////////////
const state = reactive<NekoState>({
  authenticated: false,
  connection: {
    url: location.href,
    token: undefined,
    status: 'disconnected',
    websocket: {
      connected: false,
      config: {
        max_reconnects: 15,
        timeout_ms: 5000,
        backoff_ms: 1500,
      },
    },
    webrtc: {
      connected: false,
      stable: false,
      config: {
        max_reconnects: 15,
        timeout_ms: 10000,
        backoff_ms: 1500,
      },
      stats: null,
      video: {
        disabled: false,
        id: '',
        auto: false,
      },
      audio: {
        disabled: false,
      },
      videos: [],
    },
    screencast: true, // TODO: Should get by API call.
    type: 'none',
  },
  video: {
    playable: false,
    playing: false,
    volume: 0,
    muted: false,
  },
  control: {
    scroll: {
      inverse: true,
      sensitivity: 0,
    },
    clipboard: null,
    keyboard: {
      layout: 'us',
      variant: '',
    },
    touch: {
      enabled: true,
      supported: false,
    },
    host_id: null,
    is_host: false,
    locked: false,
  },
  screen: {
    size: {
      width: 1280,
      height: 720,
      rate: 30,
    },
    configurations: [],
    sync: {
      enabled: false,
      multiplier: 0,
      rate: 30,
    },
  },
  session_id: null,
  sessions: {},
  settings: {
    private_mode: false,
    locked_logins: false,
    locked_controls: false,
    control_protection: false,
    implicit_hosting: false,
    inactive_cursors: false,
    merciful_reconnect: false,
  },
  cursors: [],
  mobile_keyboard_open: false,
  watchingSessions: {},
})

/////////////////////////////
// Connection manager
/////////////////////////////

const connection = new NekoConnection(state.connection)

const connected = computed(() => state.connection.status == 'connected')
const controlling = computed(() => state.control.host_id !== null && state.session_id === state.control.host_id)
const session = computed(() => (state.session_id != null ? state.sessions[state.session_id] : null))
const is_admin = computed(() => session.value?.profile.is_admin || false)
const private_mode_enabled = computed(() => state.settings.private_mode && !is_admin.value)
const is_touch_device = computed(() => {
  if (props.inputMode == 'mouse') return false
  if (props.inputMode == 'touch') return true

  return (
    // check if the device has a touch screen
    ('ontouchstart' in window || navigator.maxTouchPoints > 0) &&
    // we also check if the device has a pointer
    !window.matchMedia('(pointer:fine)').matches &&
    // and is capable of hover, then it probably has a mouse
    !window.matchMedia('(hover:hover)').matches
  )
})

watch(private_mode_enabled, (enabled) => {
  connection.webrtc.paused = enabled
})

const screencastReady = ref(false)
const screencast = computed(() => {
  return (
    state.authenticated &&
    state.connection.status != 'disconnected' &&
    state.connection.screencast &&
    (!state.connection.webrtc.connected || (state.connection.webrtc.connected && !state.video.playing))
  )
})

/////////////////////////////
// Public events
/////////////////////////////
const events = new NekoMessages(connection, state)

/////////////////////////////
// Public methods
/////////////////////////////

function setUrl(url?: string) {
  if (!url) {
    url = location.href
  }

  // url can contain ?token=<string>
  let token = new URL(url).searchParams.get('token') || undefined

  // get URL without query params
  url = url.split('?')[0]

  const httpURL = url.replace(/^ws/, 'http').replace(/\/$|\/ws\/?$/, '')
  api.setUrl(httpURL)
  state.connection.url = httpURL // TODO: Vue.Set

  try {
    disconnect()
  } catch {}

  if (state.authenticated) {
    state.authenticated = false // TODO: Vue.Set
  }

  // save token to state
  state.connection.token = token // TODO: Vue.Set

  // try to authenticate and connect
  if (props.autoconnect) {      
    try {
      authenticate()
    connect()
    } catch {}
  }
}

watch(() => props.server, (url) => {
  setUrl(url)
}, { immediate: true })

async function authenticate(token?: string) {
  if (!token) {
    token = state.connection.token
  }

  if (!token && props.autologin) {
    token = localStorage.getItem('neko_session') ?? undefined
  }

  if (token) {
    api.setToken(token)
    state.connection.token = token // TODO: Vue.Set
  }

    await api.default.whoami()
    state.authenticated = true // TODO: Vue.Set

  if (token && props.autologin) {
    localStorage.setItem('neko_session', token)
  }
}

async function login(username: string, password: string) {
  if (state.authenticated) {
    throw new Error('client already authenticated')
  }

      const res = await api.default.login({ username, password })
      if (res.data.token) {
        api.setToken(res.data.token)
        state.connection.token = res.data.token // TODO: Vue.Set

    if (props.autologin) {
        localStorage.setItem('neko_session', res.data.token)
      }
    }

  state.authenticated = true // TODO: Vue.Set
}

async function logout() {
  if (!state.authenticated) {
    throw new Error('client not authenticated')
  }

  try {
    disconnect()
  } catch {}

  try {
    await api.default.logout()
  } finally {
    api.setToken('')
    delete state.connection.token // TODO: Vue.Delete

    if (props.autologin) {
      localStorage.removeItem('neko_session')
    }

    state.authenticated = false // TODO: Vue.Set
  }
}

function connect(peerRequest?: webrtcTypes.PeerRequest) {
  if (!state.authenticated) {
    throw new Error('client not authenticated')
  }

  if (connected.value) {
    throw new Error('client is already connected')
  }

  connection.open(peerRequest)
}

function disconnect() {
  connection.close()
}

function setReconnectorConfig(type: 'websocket' | 'webrtc', config: ReconnectorConfig) {
  if (type != 'websocket' && type != 'webrtc') {
    throw new Error('unknown reconnector type')
  }

  state.connection[type].config = config // TODO: Vue.Set
  connection.reloadConfigs()
}

async function play() {
  // if autoplay is disabled, play() will throw an error
  // and we need to properly save the state otherwise we
  // would be thinking we're playing when we're not
  try {
    await video.value!.play()
  } catch (e: any) {
    if (video.value!.muted) {
      throw e
    }

    // video.play() can fail if audio is set due restrictive
    // browsers autoplay policy -> retry with muted audio
    try {
      video.value!.muted = true
      await video.value!.play()
      // unmute on users first interaction
      document.addEventListener('click', autoUnmute, { once: true })
      overlay.value!.once('overlay.click', autoUnmute)
    } catch (e: any) {
      // if it still fails, we're not playing anything
      video.value!.muted = false
      throw e
    }
  }
}

function pause() {
  video.value!.pause()
}

function mute() {
  video.value!.muted = true
}

function unmute() {
  video.value!.muted = false
}

// when autoplay fails, we mute the video and wait for the user
// to interact with the page to unmute it again
function autoUnmute() {
  unmute()

  // remove listeners
  document.removeEventListener('click', autoUnmute)
  overlay.value!.removeListener('overlay.click', autoUnmute)
}

function setVolume(value: number) {
  if (value < 0 || value > 1) {
    throw new Error('volume must be between 0 and 1')
  }

  video.value!.volume = value
}

function setScrollInverse(value: boolean = true) {
  state.control.scroll.inverse = value // TODO: Vue.Set
}

function setScrollSensitivity(value: number) {
  state.control.scroll.sensitivity = value // TODO: Vue.Set
}

function setKeyboard(layout: string, variant: string = '') {
  state.control.keyboard = { layout, variant } // TODO: Vue.Set
}

function setTouchEnabled(value: boolean = true) {
  state.control.touch.enabled = value // TODO: Vue.Set
}

function mobileKeyboardShow() {
  overlay.value!.mobileKeyboardShow()
}

function mobileKeyboardHide() {
  overlay.value!.mobileKeyboardHide()
}

function mobileKeyboardToggle() {
  if (state.mobile_keyboard_open) {
    mobileKeyboardHide()
  } else {
    mobileKeyboardShow()
  }
}

function setScreenSync(enabled: boolean = true, multiplier: number = 0, rate: number = 60) {
  state.screen.sync.enabled = enabled // TODO: Vue.Set
  state.screen.sync.multiplier = multiplier // TODO: Vue.Set
  state.screen.sync.rate = rate // TODO: Vue.Set
}

function setCursorDrawFunction(fn?: CursorDrawFunction) {
  cursorDrawFunction.value = (fn || null)
}

function setInactiveCursorDrawFunction(fn?: InactiveCursorDrawFunction) {
  inactiveCursorDrawFunction.value = (fn || null)
}

// TODO: Remove? Use REST API only?
function setScreenSize(width: number, height: number, rate: number) {
  connection.websocket.send(EVENT.SCREEN_SET, { width, height, rate })
}

function setWebRTCVideo(peerVideo: webrtcTypes.PeerVideoRequest) {
  connection.websocket.send(EVENT.SIGNAL_VIDEO, peerVideo)
}

function setWebRTCAudio(peerAudio: webrtcTypes.PeerAudioRequest) {
  connection.websocket.send(EVENT.SIGNAL_AUDIO, peerAudio)
}

function addTrack(track: MediaStreamTrack, ...streams: MediaStream[]): RTCRtpSender {
  return connection.webrtc.addTrack(track, ...streams)
}

function removeTrack(sender: RTCRtpSender) {
  connection.webrtc.removeTrack(sender)
}

function sendUnicast(receiver: string, subject: string, body: any) {
  connection.websocket.send(EVENT.SEND_UNICAST, { receiver, subject, body })
}

function sendBroadcast(subject: string, body: any) {
  connection.websocket.send(EVENT.SEND_BROADCAST, { subject, body })
}

function sendMessage(event: string, payload?: any) {
  connection.websocket.send(event, payload)
}

function withApi<T>(c: new (configuration?: Configuration, basePath?: string, axios?: AxiosInstance) => T): T {
  return new c(api.config)
}

const control = new NekoControl(connection, state.control)

const sessions = computed<SessionsApi>(() => api.sessions)
const room = computed<RoomApi>(() => api.room)
const members = computed<MembersApi>(() => api.members)

async function uploadDrop({ x, y, files }: { x: number; y: number; files: Array<File> }) {
  try {
    events.emit('upload.drop.started')

    await api.room.uploadDrop(x, y, files, {
      onUploadProgress: (progressEvent: AxiosProgressEvent) => {
        events.emit('upload.drop.progress', progressEvent)
      },
    })

    events.emit('upload.drop.finished')
  } catch (err: any) {
    events.emit('upload.drop.finished', err)
  }
}

/////////////////////////////
// Component lifecycle
/////////////////////////////

onMounted(() => {
  // component size change
  observer.observe(component.value!)

  // webrtc needs video tag to capture video snaps for fallback mode
  connection.webrtc.video = video.value!

  // video events
  VideoRegister(video.value!, state.video)

  connection.on('close', (error) => {
    events.emit('connection.closed', error)
    clear()
  })

  // when webrtc emits fallback event, it means it is about to reconnect
  // so we image that it provided (it is last frame of the video), we set
  // it to the screencast module and pause video in order to show fallback
  connection.webrtc.on('fallback', (image: string) => {
    fallbackImage.value = image

    // this ensures that fallback mode starts immediatly
    pause()
  })

  connection.webrtc.on('track', (event: RTCTrackEvent) => {
    const { track, streams } = event
    if (track.kind === 'audio') return

    // apply track only once it is unmuted
    track.addEventListener(
      'unmute',
      () => {
        // create stream
        if ('srcObject' in video.value!) {
          video.value.srcObject = streams[0]
        } else {
          // @ts-ignore
          video.value.src = window.URL.createObjectURL(streams[0]) // for older browsers
        }

        if (props.autoplay || connection.activated) {
          play()
        }
      },
      { once: true },
    )
  })

watch(() => state.connection.websocket.connected, (connected) => {
  if (connected) {
    getvolume();
    setVolume(volume.value / 100);
  }
});

function getvolume() {
  if (!localStorage.getItem('volume')) {
    localStorage.setItem('volume', '100')
  }
  let val = localStorage.getItem('volume') || '100';
  volume.value = parseInt(val)
  }

onBeforeUnmount(() => {
  localStorage.setItem('volume', `${volume.value}`) // save volume;
  observer.disconnect()
  connection.destroy()
  clear()

  // removes users first interaction events
  autoUnmute()
})

onUnmounted(() => {
  localStorage.setItem('volume', `${volume.value}`) // save volume;
})

function updateKeyboard() {
  if (controlling.value && state.control.keyboard.layout) {
    connection.websocket.send(EVENT.KEYBOARD_MAP, state.control.keyboard)
  }
}

watch(controlling, updateKeyboard)
watch(() => state.control.keyboard, updateKeyboard)

function updateKeyboardModifiers(modifiers: { capslock: boolean; numlock: boolean }) {
  connection.websocket.send(EVENT.KEYBOARD_MODIFIERS, modifiers)
}

function onScreenSyncChange() {
  if (state.screen.sync.enabled) {
    syncScreenSize()
    window.addEventListener('resize', syncScreenSize)
  } else {
    window.removeEventListener('resize', syncScreenSize)
  }
}

watch(() => state.screen.sync.enabled, onScreenSyncChange)

let syncScreenSizeTimeout = 0

function syncScreenSize() {
  if (syncScreenSizeTimeout) {
    window.clearTimeout(syncScreenSizeTimeout)
  }
  syncScreenSizeTimeout = window.setTimeout(() => {
    const multiplier = state.screen.sync.multiplier || window.devicePixelRatio
    syncScreenSizeTimeout = 0
    const { offsetWidth, offsetHeight } = component.value!
    setScreenSize(
      Math.round(offsetWidth * multiplier),
      Math.round(offsetHeight * multiplier),
      state.screen.sync.rate,
    )
  }, SCREEN_SYNC_THROTTLE)
}

function onResize() {
  const { width, height } = state.screen.size
  const screenRatio = width / height

  const { offsetWidth, offsetHeight } = component.value!
  const canvasRatio = offsetWidth / offsetHeight

  // vertical centering
  if (screenRatio > canvasRatio) {
    const vertical = offsetWidth / screenRatio
    container.value!.style.width = `${offsetWidth}px`
    container.value!.style.height = `${vertical}px`
    container.value!.style.marginTop = `${(offsetHeight - vertical) / 2}px`
    container.value!.style.marginLeft = `0px`

    canvasSize.value = {
      width: offsetWidth,
      height: vertical,
    }
  }
  // horizontal centering
  else if (screenRatio < canvasRatio) {
    const horizontal = screenRatio * offsetHeight
    container.value!.style.width = `${horizontal}px`
    container.value!.style.height = `${offsetHeight}px`
    container.value!.style.marginTop = `0px`
    container.value!.style.marginLeft = `${(offsetWidth - horizontal) / 2}px`

    canvasSize.value = {
      width: horizontal,
      height: offsetHeight,
    }
  }
  // no centering
  else {
    container.value!.style.width = `${offsetWidth}px`
    container.value!.style.height = `${offsetHeight}px`
    container.value!.style.marginTop = `0px`
    container.value!.style.marginLeft = `0px`

    canvasSize.value = {
      width: offsetWidth,
      height: offsetHeight,
    }
  }
}

watch(() => state.screen.size, onResize)

function updateConnectionType() {
  if (screencast.value) {
    state.connection.type = 'fallback' // TODO: Vue.Set
  } else if (state.connection.webrtc.connected) {
    state.connection.type = 'webrtc' // TODO: Vue.Set
  } else {
    state.connection.type = 'none' // TODO: Vue.Set
  }
}

watch(screencast, updateConnectionType)
watch(() => state.connection.webrtc.connected, updateConnectionType)

function onConnectionStatusChange(status: 'connected' | 'connecting' | 'disconnected') {
  events.emit('connection.status', status)
}

watch(() => state.connection.status, onConnectionStatusChange)

function onConnectionTypeChange(type: 'fallback' | 'webrtc' | 'none') {
  events.emit('connection.type', type)
}

watch(() => state.connection.type, onConnectionTypeChange)


const screenConfiguration = ref(`${state.screen.size.width}x${state.screen.size.height}@${state.screen.size.rate}`)
function setScreenConfiguration(value: string) {
  let [width, height, rate] = value.split(/[@x]/)
  setScreenSize(parseInt(width), parseInt(height), parseInt(rate))
}
function toggleDropdown() {
  showScreenConfigMenu.value = !showScreenConfigMenu.value;
}

watch(() => screenConfiguration.value, (val) => {
  if (is_admin.value) {
    setScreenConfiguration(val)
  }
})

function clear() {
  // destroy video
  if (video.value) {
    if ('srcObject' in video.value) {
      video.value.srcObject = null
    } else {
      // @ts-ignore
      video.value.removeAttribute('src')
    }
  }

  // websocket
  state.control.clipboard = null // TODO: Vue.Set
  state.control.host_id = null // TODO: Vue.Set
  state.control.is_host = false // TODO: Vue.Set
  state.screen.size = { width: 1280, height: 720, rate: 30 } // TODO: Vue.Set
  state.screen.configurations = [] // TODO: Vue.Set
  state.screen.sync.enabled = false // TODO: Vue.Set
  state.session_id = null // TODO: Vue.Set
  state.sessions = {} // TODO: Vue.Set
  state.watchingSessions = {}
  state.settings =  {
    private_mode: false,
    locked_logins: false,
    locked_controls: false,
    control_protection: false,
    implicit_hosting: false,
    inactive_cursors: false,
    merciful_reconnect: false,
  } // TODO: Vue.Set
  state.cursors = [] // TODO: Vue.Set

  // webrtc
  state.connection.webrtc.stats = null // TODO: Vue.Set
  state.connection.webrtc.video.disabled = false // TODO: Vue.Set
  state.connection.webrtc.video.id = '' // TODO: Vue.Set
  state.connection.webrtc.video.auto = false // TODO: Vue.Set
  state.connection.webrtc.audio.disabled = false // TODO: Vue.Set
  state.connection.webrtc.videos = [] // TODO: Vue.Set
  state.connection.type = 'none' // TODO: Vue.Set
}

defineExpose({
  setUrl,
  authenticate,
  login,
  logout,
  connect,
  disconnect,
  setReconnectorConfig,
  play,
  pause,
  mute,
  unmute,
  setVolume,
  setScrollInverse,
  setScrollSensitivity,
  setKeyboard,
  setTouchEnabled,
  mobileKeyboardShow,
  mobileKeyboardHide,
  mobileKeyboardToggle,
  setScreenSync,
  setCursorDrawFunction,
  setInactiveCursorDrawFunction,
  setScreenSize,
  setWebRTCVideo,
  setWebRTCAudio,
  addTrack,
  removeTrack,
  sendUnicast,
  sendBroadcast,
  sendMessage,
  withApi,
  uploadDrop,
  isFirstMount,
  showModal,
  // public state
  state,
  // computed
  connected,
  controlling,
  session,
  is_admin,
  private_mode_enabled,
  is_touch_device,
  screencast,
  // public events
  events,
  // public methods
  control,
  // public api
  sessions,
  room,
  members,
})
</script>
