<template>
  <div class="modal" v-if="props.neko.showModal">
    <div class="modal-content">
      <div>
        <div style="text-align: center" v-if="!neko.state.authenticated">
        <div class="form-group">
          <input type="text" class="form-control" placeholder="Username" v-model="username">
        </div>
        <div class="form-group">
        <input type="password" class="form-control" placeholder="Password" v-model="password">
        </div>
        <div class="form-group">
        <td><button class="btn-login" btn-login:hover @click="login()">Login</button></td>
        </div>
        </div>
        <div v-else style="text-align: center">
          <p style="padding-bottom: 10px">You are not connected to the server.</p>
          <button class="btn" btn:hover @click="connect()">Connect</button> or
          <button class="btn" btn:hover @click="logout()">Logout</button>
        </div>
      </div>
    </div>
  </div>
</template>
<style lang="scss">
@import '@/page/assets/styles/main.scss';
.modal {
    position: relative; /* Change from 'center' to 'fixed' */
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background-color: rgba(0, 0, 0, 0.5); /* Add a semi-transparent background color */
    display: flex;
    justify-content: center;
    align-items: center;
    z-index: 1000; /* Increase the z-index to make sure the modal is on top */
}

.modal-content {
    background-color: rgba(255, 255, 255, 0);
    padding: 20px;
    border-radius: 5px;
    max-width: 500px; /* Adjust the max-width as needed */
}

.logo-new {
  font-size: 30px;
  line-height: 30px;
  align-self: right;
  b {
    font-weight: 900;
  }
  }
.form-group {
    padding-top: 2%;
    padding-bottom: 2%;
    justify-content: center;
    align-items: center;
}
.form-control {
    background-color: #8e8e8e;
    align-self: center;
    font-size: 15px;
    font-weight: 500;
    color: #ffffff;
    text-align: center;
    height: 45px;
    width: 160px;
    border: 2px solid #2e2e2e;
    border-radius: 50px;
    box-shadow: none;
}

.form-control:focus {
    background-color: #e1e1e1;
    color: #2e2e2e;
    box-shadow: none;
}

.btn {
    color: #fff;
    background: $style-primary;
    font-size: 16px;
    font-weight: 500;
    letter-spacing: 1px;
    text-transform: uppercase;
    height: 45px; /* Match the height of the text box */
    padding: 10px 13px 13px; /* Adjust the horizontal padding to fit the text */
    border: none;
    border-radius: 50px;
    transition: all 0.3s ease 0s;
}
.btn:hover {
    text-shadow: 3px 3px rgba(0, 0, 0, 0.4);
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
    background-color: $style-primary;
}


.btn-login {
    color: #fff;
    background: $style-primary;
    font-size: 16px;
    font-weight: 500;
    letter-spacing: 1px;
    text-transform: uppercase;
    height: 45px; /* Match the height of the text box */
    width: 100px;
    margin-left: 30px;
    padding-top: 2%;
    border: none;
    border-radius: 50px;
    transition: all 0.3s ease 0s;
}
.btn-login-2 {
    color: #fff;
    background: #5539cc;
    font-size: 12px;
    font-weight: 500;
    letter-spacing: 1px;
    text-transform: uppercase;
    height: 45px; /* Match the height of the text box */
    width: 100px;
    margin-left: 30px;
    padding-top: 2%;
    border: none;
    border-radius: 50px;
    transition: all 0.3s ease 0s;
}
.btn-login:hover {
    text-shadow: 3px 3px rgba(0, 0, 0, 0.4);
    box-shadow: 0 0 5px rgba(0, 0, 0, 0.2);
    background-color: $style-primary;
}

</style>

<script lang="ts" setup>
import { ref, watch, defineProps, defineEmits, onMounted } from 'vue'
import type Neko from '@/component/main.vue'

const props = defineProps<{
  neko: typeof Neko
}>()


const emits = defineEmits<{
  (e: 'update:showModal', value: boolean): void
}>()


const username = ref('admin')
const password = ref('admin')

const showModalLocal = ref(props.neko.showModal)

// Watch the neko.state.authenticated and close the modal when authenticated
watch(() => props.neko.state.connected, (newValue) => {
  if (newValue) {
    showModalLocal.value = false
    emits('update:showModal', false)
  }
})

watch(showModalLocal, (newValue) => {
  emits('update:showModal', newValue)
})

async function login() {
  localStorage.setItem('username', username.value)
  localStorage.setItem('password', password.value)

  try {
    await props.neko.login(username.value, password.value)
    showModalLocal.value = false
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function connect() {
  try {
    await props.neko.connect()
    showModalLocal.value = false
  } catch (e: any) {
    alert(e)
  }
}

async function logout() {
  try {
    await props.neko.logout()
    showModalLocal.value = true
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

onMounted(() => {
  const u = localStorage.getItem('username')
  if (u) {
    username.value = u
  }

  const p = localStorage.getItem('password')
  if (p) {
    password.value = p
  }
})
</script>
