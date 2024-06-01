<template>
  <div class="header">
    <div class="neko">
      <div class="user-pfp">
      </div>
      <span class="logo"><b>n</b>.eko</span>
      <div class="server">
        <span>Server:</span>
        <input type="text" placeholder="URL" v-model="url" />
        <button @click="setUrl">change</button>
      </div>
      <ul class="menu">
        <li>
          <i class="fas fa-bars toggle" @click="toggleMenu" />
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  @import '../assets/styles/_variables.scss';


  .header {
  flex: 1;
  display: flex;
  flex-direction: row;
  align-items: center;
  height: 100%;

  /* Position menu to the right */
  .menu {
    margin-left: auto; /* Pushes the menu to the right */
    margin-right: 10px; /* Optional: Adds spacing from the right edge */
  }

  .neko {
    flex: 1;
    display: flex;
    align-items: center;
    width: 150px;
    margin-left: 20px;

    .logo {
      font-size: 30px;
      line-height: 30px;
      // Remove any margin-right that was here

      b {
        font-weight: 900;
      }
    }
    .user-pfp {
        width: 30px;
        height: 30px;
        margin-top: 2px;
        padding-right: 1%;
        flex-shrink: 0; // Prevent the profile picture from shrinking


        img {
          width: 100%;
          height: 100%;
          border-radius: 50%; // Makes the pfp circular
        }
      }
      .server {
        max-width: 850px;
        width: 100%;
        margin: 0 20px;
        display: flex;
        align-items: center;

        input {
          margin: 0 5px;
          width: 100%;
        }
      }
    .menu {
      justify-self: flex-end;
      margin-right: 10px;
      white-space: nowrap;

      li {
        display: inline-block;
        margin-right: 10px;

        i {
          display: block;
          width: 30px;
          height: 30px;
          text-align: center;
          line-height: 32px;
          border-radius: 3px;
          cursor: pointer;
        }

        .toggle {
          background: $background-primary;
        }
      }
    }
  }
}
</style>

<script lang="ts" setup>
import { ref, watch } from 'vue'
import type Neko from '@/component/main.vue'

const props = defineProps<{
  neko: typeof Neko
}>()

const url = ref('')

watch(() => props.neko.state.connection.url, (u) => {
  url.value = u
})

async function setUrl() {
  if (url.value == '') {
    url.value = location.href
  }

  await props.neko.setUrl(url.value)
}

const emits = defineEmits(['toggle'])
function toggleMenu() {
  //props.neko.toggleSide()
  emits('toggle');
}
</script>
