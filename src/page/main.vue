<template>
  <div id="neko" :class="[expanded ? 'expanded' : '']">
    <main class="neko-main">
      <div class="header-container" v-if="neko">
        <NekoHeader :neko="neko" @toggle="expanded = !expanded" />
      </div>
      <div class="video-container">
        <NekoConnect v-if="neko && neko!.state.connection.status == 'disconnected'" :neko="neko" />
        <NekoCanvas ref="neko" :server="server" autologin autoconnect autoplay />
        <div v-if="loaded && neko!.private_mode_enabled" class="player-notif">Private mode is currently enabled.</div>
        <div
          v-if="loaded && neko!.state.connection.type === 'webrtc' && !neko!.state.video.playing"
          class="player-overlay"
        >
          <i @click.stop.prevent="neko!.play()" v-if="neko!.state.video.playable" class="fas fa-play-circle" />
        </div>
        <div v-if="uploadActive" class="player-overlay" style="background: rgba(0, 0, 0, 0.8); font-size: 1vw">
          UPLOAD IN PROGRESS: {{ Math.round(uploadProgress) }}%
        </div>
        <div
          v-else-if="dialogOverlayActive"
          class="player-overlay"
          style="background: rgba(0, 0, 0, 0.8); font-size: 1vw"
        >
          SOMEONE IS UPLOADING A FILE, PLEASE WAIT
        </div>
        <div
          v-else-if="dialogRequestActive"
          class="player-overlay"
          style="background: rgba(0, 0, 0, 0.8); font-size: 1vw; flex-flow: column"
          @dragenter.stop.prevent
          @dragleave.stop.prevent
          @dragover.stop.prevent
          @drop.stop.prevent="dialogUploadFiles([...$event.dataTransfer!.files])"
        >
          <span style="padding: 1em">UPLOAD REQUESTED:</span>
          <span style="background: white">
            <input type="file" @change="dialogUploadFiles([...($event.target as HTMLInputElement)!.files!])" multiple />
          </span>
          <span style="padding: 1em; padding-bottom: 0; font-style: italic">(or drop files here)</span>
          <span style="padding: 1em">
            <button @click="dialogCancel()">CANCEL</button>
          </span>
        </div>
      </div>
      <div class="room-container" style="text-align: center">
        <button
          v-if="loaded && isTouchDevice"
          @click="neko!.mobileKeyboardToggle"
          style="position: absolute; left: 5px; transform: translateY(-100%)"
          >
          <i class="fa fa-keyboard" />
        </button>
        <!-- <span v-if="loaded && neko!.state.session_id" style="padding-top: 10px">
          You are logged in as
          <strong style="font-weight: bold">
            {{ neko!.state.sessions[neko!.state.session_id].profile.name }}
          </strong>
        </span> -->

        <div class="room-menu">
          <div class="controls-container">
            <div class="left-menu">
              <i
                @click.left="cursorEnabled = !cursorEnabled"
                :class="{'fas fa-solid fa-computer-mouse': true, 'active-cursor': cursorEnabled }"
                v-b-tooltip.hover title="Toggle Native Mouse"
              />
            </div>
            <div class="controls">
              <template v-if="loaded && neko">
                <NekoControls :neko="neko" />
              </template>
            </div>
            <div class="right-menu">
              <div style="text-align: right" v-if="loaded">
                <i class="fas fa-sign-out-alt" @click="neko!.logout()" v-if="neko!.state.connection.status != 'disconnected'" v-b-tooltip.hover title="Sign Out"/>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>
    <aside class="neko-menu" v-if="expanded">
      <div class="tabs-container" v-if="neko">
        <ul>
          <li :class="{ active: tab === 'chat' }" @click.prevent="tab = 'chat'">
            <i class="fas fa-comment-alt" />
            <span v-show="tab === 'chat'">Chat</span>
          </li>
          <li :class="{ active: tab === 'members' }" @click.prevent="tab = 'members'" v-if="neko.is_admin">
            <i class="fas fa-users" />
            <span v-show="tab === 'members'">Members</span>
          </li>
          <li :class="{ active: tab === 'events' }" @click.prevent="tab = 'events'" v-if="neko.is_admin">
            <i class="fas fa-sliders-h" />
            <span v-show="tab === 'events'">Settings</span>
          </li>
          <li :class="{ active: tab === 'media' }" @click.prevent="tab = 'media'" v-if="neko.is_admin">
            <i class="fas fa-microphone" />
            <span v-show="tab === 'media'">Media</span>
          </li>

          <!-- Plugins -->
          <component v-for="(el, key) in pluginsTabs" :key="key" :is="el" :tab="tab" @tab="tab = $event" />
        </ul>
      </div>
      <div class="page-container" v-if="neko">
        <keep-alive>
          <NekoMembers  v-if="neko && (tab === 'chat' || tab === 'members' || tab === 'events' || tab === 'media' || tab === 'filetransfer')" :neko="neko" :hideNewMemberForm="tab !== 'members'" class="show-border"/>
        </keep-alive>
        <keep-alive>
          <NekoChat     :neko="neko" v-if="neko && tab === 'chat'"/>
        </keep-alive>
        <NekoEvents   v-if="neko && tab === 'events' && neko.is_admin" :neko="neko"/>
        <NekoMedia    v-if="neko && tab === 'media' && neko.is_admin" :neko="neko"/>
        <!-- Plugins -->
        <component v-for="(el, key) in pluginsComponents" :key="key" :is="el" :tab="tab" :neko="neko"/>
      </div>
    </aside>
  </div>
</template>

<style lang="scss">
  @import '@/page/assets/styles/main.scss';

  .video-container {
    position: relative;
    overflow: hidden;
    width: 100%;
    height: 100%;
  }

  .player-notif {
    position: absolute;
    top: 0;
    overflow: hidden;
    background: #2a5f2a;
    padding: 10px;
  }

  .player-overlay {
    position: absolute;
    top: 0;
    bottom: 0;
    width: 100%;
    height: 100%;
    overflow: hidden;

    background: rgba($color: #000, $alpha: 0.2);
    display: flex;
    justify-content: center;
    align-items: center;

    i {
      cursor: pointer;
      &::before {
        font-size: 120px;
        text-align: center;
      }
    }

    &.hidden {
      display: none;
    }
  }

  #neko {
    position: absolute;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    max-width: 100vw;
    max-height: 100vh;
    flex-direction: row;
    display: flex;
  }

  .neko-main {
    min-width: 360px;
    max-width: 100%;
    flex: 1;
    flex-direction: column;
    display: flex;
    overflow: auto;

    .header-container {
      background: $background-tertiary;
      height: $menu-height;
      flex-shrink: 0;
    }

    .video-container {
      background: rgba($color: #000, $alpha: 0.4);
      max-width: 100%;
      flex-grow: 1;
    }

    .room-container {
      $offset: translateX(calc(100% - $side-width));
      position: relative;
       // right: $offset;
      width: $offset;
      // position: relative;
      background: $background-tertiary;
      height: $controls-height;
      max-width: $offset;
      flex-shrink: 0;
      flex-direction: column;
      display: flex;
      /* for mobile */
      overflow-y: scroll;
      overflow-x: auto;

      .room-menu {
        flex: 1;
        display: flex;
        justify-content: space-between; // Distribute items

        .controls-container {
          position: relative;
          margin-left: 10px;
          flex: 1;
          justify-content: flex-start;
          align-items: center;
          display: flex;
          max-width: calc(100% - neko-menu-width); // Adjust width to fit the menu
          margin-right: neko-menu-width; // Adjust margin to fit the menu
          overflow-x: scroll;

          .left-menu {
            margin-left: 10px;
            flex: 1;
            justify-content: flex-start;
            align-items: center;
            width: fit-content;
            max-width: fit-content;
          }

          .controls {
            flex: 1;
            justify-content: center;
            align-items: center;
            flex-grow: 1;
            padding-left: 10%;
            padding-right: 10%;
          }

          .right-menu {
            margin-right: 10px;
            flex: 1;
            justify-content: flex-end;
            align-items: center;
            width: fit-content;
            max-width: fit-content;
          }
        }
      }
    }
  }

  .neko-menu {
    width: $side-width;
    background-color: $background-primary;
    flex-shrink: 0;
    max-height: 100%;
    max-width: 100%;
    display: flex;
    flex-direction: column;


    .tabs-container {
      background: $background-tertiary;
      height: $menu-height;
      max-height: 100%;
      max-width: 100%;
      display: flex;
      flex-shrink: 0;

      ul {
        display: inline-block;
        padding: 16px 0 0 0;
        overflow-x: auto;

        li {
          background: $background-secondary;
          border-radius: 3px 3px 0 0;
          border-bottom: none;
          display: inline-block;
          padding: 5px 10px;
          margin-right: 4px;
          font-weight: 600;
          cursor: pointer;

          i {
            margin-right: 4px;
            font-size: 10px;
          }

          &.active {
            background: $background-primary;
          }
        }
      }
    }
    .fa-computer-mouse {
          color: $style-primary;
        }
    .switch {
        margin: 0 5px;
        display: block;
        position: relative;
        width: 42px;
        height: 24px;
        input[type='checkbox'] {
          opacity: 0;
          width: 0;
          height: 0;
        }
        span {
          position: absolute;
          cursor: pointer;
          top: 0;
          left: 0;
          right: 0;
          bottom: 0;
          background-color: $background-secondary;
          transition: 0.2s;
          border-radius: 34px;
          &:before {
            color: $background-tertiary;
            font-weight: 900;
            font-family: 'Font Awesome 6 Free';
            content: '\f3c1';
            font-size: 8px;
            line-height: 18px;
            text-align: center;
            position: absolute;
            height: 18px;
            width: 18px;
            left: 3px;
            bottom: 3px;
            background-color: white;
            transition: 0.3s;
            border-radius: 50%;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.3);
          }
        }
      }
      input[type='checkbox'] {
        &:checked + span {
          background-color: $style-primary;
          &:before {
            content: '\f023';
            transform: translateX(18px);
          }
        }
        &:disabled + span {
          &:before {
            content: '';
            background-color: rgba($color: $text-normal, $alpha: 0.4);
          }
        }
    }

    .page-container-wrapper {
      display: flex;
      flex-direction: column;
      justify-content: space-between; /* This spreads out the child elements */
      height: 100%;
    }

    .scrollable-members {
      flex-grow: 0;
      flex-shrink: 0;
      max-height: 75%;
      overflow-y: auto;
      padding: 5px;
      box-sizing: border-box;
      width: 100%; /* Ensure it takes full width */
    }

    .chat-container {
      flex-grow: 0;
      flex-shrink: 0;
      padding: 5px;
      overflow-y: auto;
      box-sizing: border-box;
      width: 100%; /* Ensure it takes full width */
    }
    // .page-container {
    //   // max-height: 100%;
    //   // flex-grow: 1;
    //   // display: flex;
    //   // flex-direction: column;
    //   // hyphens: auto;
    //   // overflow-y: auto;
    //   // padding: 5px;
    //   // box-sizing: border-box;
    //   // border-bottom: 1px solid rgba(0, 0, 0, 0.41); /* Add this line */
    //   .page-container .below-members {
    //     flex-grow: 0;
    //     flex-shrink: 0;
    //     padding: 5px;
    //     overflow-y: auto;
    //     box-sizing: border-box;
    //     width: 100%; /* Ensure it takes full width */
    //   }
    // }
  }
  .page-container {
      max-height: 100%;
      flex-grow: 1;
      display: flex;
      flex-direction: column;
      hyphens: auto;
      overflow-y: auto;
      padding: 5px;
      box-sizing: border-box;
      border-bottom: 1px solid rgba(0, 0, 0, 0.41); /* Add this line */
      .below-members {
        flex-grow: 0;
        flex-shrink: 0;
        padding: 5px;
        overflow-y: auto;
        box-sizing: border-box;
        width: 100%; /* Ensure it takes full width */
      }
      .show-border {
        border-bottom: 3px solid rgba(0, 0, 0, 0.3);
        border-radius: 10px;
        background: rgba(0, 0, 0, 0.3);
      }
  }


  /* for mobile */
  @media only screen and (max-width: 600px) {
    $offset: 38px;

    #neko.expanded {
      /* show only enough of the menu to see the toggle button */
      .neko-main {
        transform: translateX(calc(-100% + $offset));
        video {
          display: none;
        }
      }
      .neko-menu {
        position: absolute;
        top: 0;
        right: 0;
        bottom: 0;
        left: $offset;
        width: calc(100% - $offset);
      }
      /* display menu toggle button far right */
      .header .menu,
      .header .menu li {
        margin-right: 2px;
      }
    }
  }
  .active-cursor {
  color: $style-primary;
}

.fa-computer-mouse {
  cursor: pointer;
}

.fa-sign-out-alt {
  cursor: pointer;
}

.room-menu {
  // ... other styles

  .left-menu,
  .right-menu {
    font-size: 1.8em; // Increase font size by 20%
    margin: 0 15px;   // Adjust margin to move closer to center

  }
}

</style>

<script lang="ts" setup>
// plugins must be available at:
// ./plugins/{name}/index.ts -> { Components, Tabs }
const plugins = import.meta.glob('./plugins/*/index.ts')

const pluginsTabs = shallowRef<Record<string, any>>({})
const pluginsComponents = shallowRef<Record<string, any>>({})

// dynamic plugins loader
onMounted(async () => {
  const resolvedPlugins = await Promise.all(
    Object.entries(plugins).map(async ([path, component]) => {
      return [path, await component()]
    }),
  ) as [string, { Components: any, Tabs: any }][]

  pluginsTabs.value = {}
  pluginsComponents.value = {}
  for (const [path, { Components, Tabs }] of resolvedPlugins) {
    pluginsTabs.value[path] = Tabs
    pluginsComponents.value[path] = Components
  }
})

import { ref, shallowRef, computed, onMounted, watch } from 'vue'

import type { AxiosProgressEvent } from 'axios'
import NekoCanvas from '@/component/main.vue'
import type { Settings } from '@/component/types/state'
import NekoHeader from './components/header.vue'
import NekoConnect from './components/connect.vue'
import NekoControls from './components/controls.vue'
import NekoEvents from './components/events.vue'
import NekoMembers from './components/members.vue'
import NekoMedia from './components/media.vue'
import NekoChat from './components/chat.vue'

const neko = ref<typeof NekoCanvas>()
const cursorEnabled = ref<boolean>(true)


const expanded = ref(!window.matchMedia('(max-width: 600px)').matches) // default to expanded on bigger screens
const loaded = ref(false)
const tab = ref('')

const server = ref(location.href)

const uploadActive = ref(false)
const uploadProgress = ref(0)

const isTouchDevice = computed(() => 'ontouchstart' in window || navigator.maxTouchPoints > 0)

const dialogOverlayActive = ref(false)
const dialogRequestActive = ref(false)
async function dialogUploadFiles(files: File[]) {
  console.log('will upload files', files)

  uploadActive.value = true
  uploadProgress.value = 0
  try {
    await neko.value!.room.uploadDialog([...files], {
      onUploadProgress: (progressEvent: AxiosProgressEvent) => {
        if (!progressEvent.total) {
          uploadProgress.value = 0
          return
        }
        uploadProgress.value = (progressEvent.loaded / progressEvent.total) * 100
      },
    })
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  } finally {
    uploadActive.value = false
  }
}

const usesCursor = ref(false)

function dialogCancel() {
  neko.value!.room.uploadDialogClose()
}

// Watch for changes in cursorEnabled
watch(cursorEnabled, (isEnabled) => {
  if (isEnabled) {
    enableCursor()
    console.log('cursor enabled')
  } else {
    // disableCursor()
    disableCursor()
    console.log('cursor disabled')
  }
});



onMounted(() => {
  loaded.value = true
  tab.value = 'chat'
  //@ts-ignore
  window.neko = neko

  // initial URL
  const url = new URL(location.href).searchParams.get('url')
  if (url) {
    server.value = url
  }
  neko.value!.events.on('connection.status', () => {
    if (cursorEnabled.value) {
      enableCursor();
    }
  })

  //
  // connection events
  //
  neko.value!.events.on('connection.status', (status: 'connected' | 'connecting' | 'disconnected') => {
    console.log('connection.status', status)
  })
  neko.value!.events.on('connection.type', (type: 'fallback' | 'webrtc' | 'none') => {
    console.log('connection.type', type)
  })
  neko.value!.events.on('connection.webrtc.sdp', (type: 'local' | 'remote', data: string) => {
    console.log('connection.webrtc.sdp', type, data)
  })
  neko.value!.events.on('connection.webrtc.sdp.candidate', (type: 'local' | 'remote', data: RTCIceCandidateInit) => {
    console.log('connection.webrtc.sdp.candidate', type, data)
  })
  neko.value!.events.on('connection.closed', (error?: Error) => {
    if (error) {
      alert('Connection closed with error: ' + error.message)
    } else {
      console.log('Connection closed without error.')
    }
  })

  //
  // drag and drop events
  //
  neko.value!.events.on('upload.drop.started', () => {
    uploadActive.value = true
    uploadProgress.value = 0
  })
  neko.value!.events.on('upload.drop.progress', (progressEvent: AxiosProgressEvent) => {
    if (!progressEvent.total) {
      uploadProgress.value = 0
      return
    }
    uploadProgress.value = (progressEvent.loaded / progressEvent.total) * 100
  })
  neko.value!.events.on('upload.drop.finished', (e?: any) => {
    uploadActive.value = false
    if (e) {
      alert(e.response ? e.response.data.message : e)
    }
  })

  //
  // upload dialog events
  //
  neko.value!.events.on('upload.dialog.requested', () => {
    dialogRequestActive.value = true
  })
  neko.value!.events.on('upload.dialog.overlay', (id: string) => {
    dialogOverlayActive.value = true
    console.log('upload.dialog.overlay', id)
  })
  neko.value!.events.on('upload.dialog.closed', () => {
    dialogOverlayActive.value = false
    dialogRequestActive.value = false
  })

  //
  // custom messages events
  //
  neko.value!.events.on('receive.unicast', (sender: string, subject: string, body: string) => {
    console.log('receive.unicast', sender, subject, body)
  })

  neko.value!.events.on('receive.broadcast', (sender: string, subject: string, body: string) => {
    console.log('receive.broadcast', sender, subject, body)
  })

  //
  // session events
  //
  neko.value!.events.on('session.created', (id: string) => {
    console.log('session.created', id)
  })
  neko.value!.events.on('session.deleted', (id: string) => {
    console.log('session.deleted', id)
  })
  neko.value!.events.on('session.updated', (id: string) => {
    console.log('session.updated', id)
  })

  //
  // room events
  //
  neko.value!.events.on('room.control.host', (hasHost: boolean, hostID: string | undefined, id: string) => {
    console.log('room.control.host', hasHost, hostID, 'by', id)
  })
  neko.value!.events.on('room.screen.updated', (width: number, height: number, rate: number, id: string) => {
    console.log('room.screen.updated', width, height, rate, 'by', id)
  })
  neko.value!.events.on('room.clipboard.updated', (text: string) => {
    console.log('room.clipboard.updated', text)
  })
  neko.value!.events.on('room.settings.updated', (settings: Settings, id: string) => {
    console.log('room.settings.updated', settings, 'by', id)
  })
  neko.value!.events.on('room.broadcast.status', (isActive: boolean, url?: string) => {
    console.log('room.broadcast.status', isActive, url)
  })

  //
  // control events
  //
  neko.value!.control.on('overlay.click', (e: MouseEvent) => {
    console.log('control: overlay.click', e)
  })
  neko.value!.control.on('overlay.contextmenu', (e: MouseEvent) => {
    console.log('control: overlay.contextmenu', e)
  })})


const pastelColors = [
  '#FFB3BA', // Light Red
  '#FFDFBA', // Light Orange
  '#FFFFBA', // Light Yellow
  '#BAFFC9', // Light Green
  '#BAE1FF', // Light Blue
  '#D1BAFF', // Light Purple
  '#FFCCF9'  // Bright Pink
];

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

function enableCursor() {
  neko.value!.setInactiveCursorDrawFunction(
    (ctx: CanvasRenderingContext2D, x: number, y: number, sessionId: string) => {
      const cursorTag = neko.value!.state.sessions[sessionId]?.profile.name || ''
      const color = getUserColor(sessionId || '')
      const colorLight = color
      const colorDark = '#000000'

      // get current cursor position
      x -= 4
      y -= 4

      // draw arrow path
      const arrowPath = new Path2D('M5 5L20 20L14 20L16 25L12 26L10 20L5 25Z')
      ctx.save(); // Save the current state
      ctx.globalAlpha = 0.75
      ctx.translate(x, y)
      ctx.scale(1.25, 1.25); // Scale the drawing context
      ctx.fillStyle = colorLight
      ctx.fill(arrowPath)
      ctx.lineWidth = 1.5
      ctx.lineJoin = 'miter'
      ctx.miterLimit = 50
      ctx.lineCap = 'round'
      ctx.strokeStyle = colorDark
      ctx.stroke(arrowPath); // Draw the path

      // draw cursor tag
      if (cursorTag) {
        const x = 22 // box margin x
        const y = 28 // box margin y

        ctx.font = '12px Arial, sans-serif'
        ctx.textBaseline = 'top'
        ctx.shadowColor = 'black'
        ctx.shadowBlur = 2
        ctx.lineWidth = 2
        ctx.fillStyle = '#000000c0'
        ctx.strokeText(cursorTag, x, y)
        ctx.shadowBlur = 0
        ctx.fillStyle = '#ffffffc0'
        ctx.fillText(cursorTag, x, y)
      }
      ctx.restore(); // Restore the state
    },
  )
  neko.value!.setCursorDrawFunction(
    (ctx: CanvasRenderingContext2D, x: number, y: number, sessionId: string) => {
      const cursorTag = neko.value!.state.sessions[sessionId]?.profile.name || ''
      const color = getUserColor(sessionId || '')
      const colorLight = color
      const colorDark = '#000000'
      const fontColor = '#ffffff'

      // get current cursor position
      x -= 4
      y -= 4

      // draw arrow path
      const arrowPath = new Path2D('M5 5L20 20L14 20L16 25L12 26L10 20L5 25Z')
      ctx.save(); // Save the current state
      ctx.globalAlpha = 1
      ctx.translate(x, y)
      ctx.scale(1.25, 1.25); // Scale the drawing context
      ctx.fillStyle = colorLight
      ctx.fill(arrowPath)
      ctx.lineWidth = 1.5
      ctx.lineJoin = 'miter'
      ctx.miterLimit = 50
      ctx.lineCap = 'round'
      ctx.strokeStyle = colorDark
      ctx.stroke(arrowPath); // Draw the path

      // draw cursor tag
      if (cursorTag) {
        const x = 22 // box margin x
        const y = 28 // box margin y

        ctx.font = '12px Arial, sans-serif'
        ctx.textBaseline = 'top'
        ctx.shadowColor = 'black'
        ctx.shadowBlur = 2
        ctx.lineWidth = 2
        ctx.fillStyle = '#000000'
        ctx.strokeText(cursorTag, x, y)
        ctx.shadowBlur = 0
        ctx.fillStyle = '#ffffff'
        ctx.fillText(cursorTag, x, y)
      }
      ctx.restore(); // Restore the state
    },
  )
}

function disableCursor() {
  // Clear both active and inactive cursor functions
  neko.value!.setCursorDrawFunction()
  neko.value!.setInactiveCursorDrawFunction(
    (ctx: CanvasRenderingContext2D, x: number, y: number, sessionId: string) => {
      const cursorTag = neko.value!.state.sessions[sessionId]?.profile.name || ''
      const colorLight = '#ffffff00'
      const colorDark = '#000000'

      // get current cursor position
      x -= 4
      y -= 4

      // draw arrow path
      const arrowPath = new Path2D('M5 5L20 20L14 20L16 25L12 26L10 20L5 25Z')
      ctx.save(); // Save the current state
      ctx.globalAlpha = 0.5
      ctx.translate(x, y)
      ctx.scale(1.25, 1.25); // Scale the drawing context
      ctx.fillStyle = colorLight
      ctx.fill(arrowPath)
      ctx.lineWidth = 1.5
      ctx.lineJoin = 'miter'
      ctx.miterLimit = 50
      ctx.lineCap = 'round'
      ctx.strokeStyle = colorDark
      ctx.stroke(arrowPath); // Draw the path

      // draw cursor tag
      if (cursorTag) {
        const x = 22 // box margin x
        const y = 28 // box margin y

        ctx.font = '12px Arial, sans-serif'
        ctx.textBaseline = 'top'
        ctx.shadowColor = 'black'
        ctx.shadowBlur = 2
        ctx.lineWidth = 2
        ctx.fillStyle = '#00000080'
        ctx.strokeText(cursorTag, x, y)
        ctx.shadowBlur = 0
        ctx.fillStyle = '#ffffffc0'
        ctx.fillText(cursorTag, x, y)
      }
      ctx.restore(); // Restore the state
    },
  )
}

</script>

<style>
  @import url('https://fonts.googleapis.com/css2?family=Roboto:wght@400;500&display=swap');
</style>
