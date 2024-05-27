<template>
  <div class="spacer"></div>
  <div class="container">
    <table class="controls-table">
    <div class="spacer"></div>
    <div class="main-controls">
      <table>
      <summary>Controls</summary>
      <ul>
        <li>
          <i
            :class="[!can_host ? 'disabled' : '', !hosting ? 'faded' : '', 'fas', 'fa-keyboard', 'request']"
            @click.stop.prevent="toggleControl"
            v-b-tooltip.hover title="Request Control"
          />
        </li>
        <li>
          <label class="switch">
            <input type="checkbox" v-model="locked" />
            <span />
          </label>
        </li>
        <li>
          <i
            :class="[{ disabled: !playable }, playing ? 'fa-pause-circle' : 'fa-play-circle', 'fas', 'play']"
            @click.stop.prevent="toggleMedia"
            v-b-tooltip.hover title="Toggle Media"
          />
        </li>
        <li>
          <div class="volume">
            <i
              :class="[volume === 0 || muted ? 'fa-volume-mute' : 'fa-volume-up', 'fas']"
              @click.stop.prevent="toggleMute"
              v-b-tooltip.hover title="Toggle Mute"
            />
            <input type="range" min="0" max="100" v-model="volume"/>
          </div>
        </li>
      </ul>
      </table>
    </div>
    <div class="chart">
      <table class="states" v-if="neko.state.connection.webrtc.stats != null">
        <tr>
        <td
          colspan="2"
          style="background: green; text-align: center"
          v-if="neko.state.connection.webrtc.stats.paused"
        >
          webrtc is paused
        </td>
        <td
          colspan="2"
          style="background: darkviolet; text-align: center"
          v-else-if="neko.state.connection.webrtc.video.disabled"
        >
          video is disabled
        </td>
        <td
          colspan="2"
          style="background: red; text-align: center"
          v-else-if="!neko.state.connection.webrtc.stats.fps"
        >
          frame rate is zero
        </td>
        <td colspan="2" style="font-weight: bold;" v-else>
          {{
            neko.state.connection.webrtc.stats.width +
            'x' +
            neko.state.connection.webrtc.stats.height +
            '@' +
            Math.floor(neko.state.connection.webrtc.stats.fps * 100) / 100
          }}
        </td>
      </tr>
      <tr>
        <th style="width: 100%; text-align: left; font-weight: bold ">Bitrate:</th>
        <td style="width: 100%; text-align: right">
          {{ Math.floor(neko.state.connection.webrtc.stats.bitrate / 1024) }}
          Kb/s
        </td>
      </tr>
      <tr>
        <th style="width: 100%; text-align: left; font-weight: bold">Latency:</th>
        <td
          :title="
            'request: ' +
            neko.state.connection.webrtc.stats.requestLatency +
            'ms, response: ' +
            neko.state.connection.webrtc.stats.responseLatency +
            'ms'
          "
          style="width: 100%; text-align: right"
        >
          {{ neko.state.connection.webrtc.stats.latency }}ms
        </td>
      </tr>
    </table>
    </div>
    <div class="spacer"></div>
    </table>
  </div>
  <div class="spacer"></div>
</template>

<style lang="scss" scoped>
  @import '../assets/styles/_variables.scss';

  ul {
    display: flex;
    flex-direction: row;
    justify-content: flex-start;
    align-items: center;
    list-style: none;
    overflow-x: scroll;
    li {
      font-size: 24px;
      cursor: pointer;
      i {
        padding: 0 5px;
        &.faded {
          color: rgba($text-normal, 0.4);
        }
        &.disabled {
          color: rgba($style-error, 0.4);

        }
      }
      .volume {
        white-space: nowrap;
        display: block;
        display: flex;
        flex-direction: row;
        justify-content: center;
        align-items: center;
        list-style: none;
        input[type='range'] {
          width: 100%;
          background: transparent;
          width: 150px;
          height: 20px;
          accent-color : $style-primary;
          &::-moz-range-thumb {
            height: 12px;
            width: 12px;
            border-radius: 12px;
            background: rgba(65, 65, 65, 0.358);
            cursor: pointer;
          }
          &::-moz-range-track {
            width: 100%;
            height: 4px;
            cursor: pointer;
            background: $style-primary;
            border-radius: 2px;
          }
          &::-webkit-slider-thumb {
            -webkit-appearance: none;
            width: 15px;
            height: 15px;
            background: $style-primary;
            border-radius: 50%;
            cursor: pointer;
            margin-top: -6px;
            ::-webkit-slider-thumb:hover {
              background: rgba(0, 0, 0, 0);
              accent-color: $style-primary;
            }
          }
          &::-webkit-slider-runnable-track {
            width: 100%;
            height: 4px;
            cursor: pointer;
            background: rgba(0, 0, 0, 0)lemonchiffon;
            border-radius: 50%;
            accent-color: $style-primary;
            ::-webkit-slider-runnable-track:hover {
              background: rgba(0, 0, 0, 0);
              accent-color: $style-primary;
            }
          }
          &:hover {
            background: rgba(0, 0, 0, 0);
            accent-color: $style-primary;
            &::-moz-range-thumb {
              background: rgba(0, 0, 0, 0);
              accent-color: $style-primary;
            }
            &::-moz-range-track {
              background: rgba(0, 0, 0, 0);
              accent-color: $style-primary;
            }
          }
        }
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
    }
  }
  .container {
    display: flex;
    flex-direction: column; /* Arrange items vertically */
    align-items: stretch; /* Make sure all items stretch to fill the container */
    justify-content: flex-start; /* Align items at the start */
    height: 100%; /* Make sure the container fills the parent's height */

    .controls-table {
      display: flex; /* Flex container */
      flex-direction: row; /* Arrange children horizontally */
      justify-content: space-between; /* Distribute space between items */
      flex-grow: 1; /* Grow to fill the remaining space */
      width: 100%; /* Make sure it fills the container's width */
    }
  
    .admin-controls,
    .main-controls,
    .chart {
      flex: 1; /* Make items grow evenly to fill space */
    }

    .chart {
      font-size: 1.2em; // Increase font size by 20%
      margin-top: 0.5%;
      max-width: 200px;
    }

    .admin-controls table summary,
    .main-controls table summary {
      font-weight: bold;
      margin-bottom: 10px; /* Adjust spacing */
      padding-bottom: 10px; /* Adjust spacing */
      display: inline-block; /* Enable margin to work properly */
    }
  }



.active-cursor {
  color: $style-primary; 
}
.fa-angles-down {
  color: $style-primary;
}
.spacer {
  flex: 1; /* this will make the spacer fill the remaining space */
}
.states td {
  white-space: nowrap; /* Prevent line breaks inside this element */
}
.admin-controls .sync {
  white-space: nowrap;
  display: block;
  display: flex;
  flex-direction: row;
  justify-content: center;
  align-items: center;
  list-style: none;
  padding-right: 3px;
  input[type='range'] {
    width: 100%;
    background: transparent;
    width: 75px;
    height: 20px;
    accent-color : $style-primary;
    &::-moz-range-thumb {
      height: 12px;
      width: 12px;
      border-radius: 12px;
      background: rgba(65, 65, 65, 0.358);
      cursor: pointer;
    }
    &::-moz-range-track {
      width: 100%;
      height: 4px;
      cursor: pointer;
      background: $style-primary;
      border-radius: 2px;
    }
    &::-webkit-slider-thumb {
      -webkit-appearance: none;
      width: 15px;
      height: 15px;
      background: $style-primary;
      border-radius: 50%;
      cursor: pointer;
      margin-top: -6px;
      ::-webkit-slider-thumb:hover {
        background: rgba(0, 0, 0, 0);
        accent-color: $style-primary;
      }
    }
    &::-webkit-slider-runnable-track {
      width: 100%;
      height: 4px;
      cursor: pointer;
      background: rgba(0, 0, 0, 0)lemonchiffon;
      border-radius: 50%;
      accent-color: $style-primary;
      ::-webkit-slider-runnable-track:hover {
        background: rgba(0, 0, 0, 0);
        accent-color: $style-primary;
      }
    }
    &:hover {
      background: rgba(0, 0, 0, 0);
      accent-color: $style-primary;
      &::-moz-range-thumb {
        background: rgba(0, 0, 0, 0);
        accent-color: $style-primary;
      }
      &::-moz-range-track {
        background: rgba(0, 0, 0, 0);
        accent-color: $style-primary;
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

const can_host = computed(() => props.neko.connected)
const hosting = computed(() => props.neko.controlling)
const volume = computed({
  get: () => props.neko.state.video.volume * 100,
  set: (volume: number) => {
    props.neko.setVolume(volume / 100)
  },
})
const muted = computed(() => props.neko.state.video.muted || props.neko.state.video.volume === 0)
const playing = computed(() => props.neko.state.video.playing)
const playable = computed(() => props.neko.state.video.playable)
const locked = computed({
  get: () => props.neko.state.control.locked,
  set: (lock: boolean) => {
    if (lock) {
      props.neko.control.lock()
    } else {
      props.neko.control.unlock()
    }
  },
})

function toggleControl() {
  if (can_host.value && hosting.value) {
    props.neko.room.controlRelease()
  }

  if (can_host.value && !hosting.value) {
    props.neko.room.controlRequest()
  }
}

function toggleMedia() {
  if (playable.value && playing.value) {
    props.neko.pause()
  }

  if (playable.value && !playing.value) {
    props.neko.play()
  }
}

function toggleMute() {
  if (playable.value && muted.value) {
    props.neko.unmute()
  }

  if (playable.value && !muted.value) {
    props.neko.mute()
  }
}

function disconnect() {
  props.neko.logout()
}
</script>
