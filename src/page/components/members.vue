<template>
  <div class="members">
    <table class="plugins" v-if="plugins">
      <tr>
        <td colspan="2" class="name">Plugins for {{ plugins.profile.name }}</td>
      </tr>
      <tr v-for="([key], i) in plugins.old" :key="key">
        <th>{{ key }}</th>
        <td><input type="text" v-model="plugins.old[i][1]" placeholder="value (JSON)" /></td>
      </tr>
      <tr v-for="([key], i) in plugins.new" :key="key">
        <th><input type="text" v-model="plugins.new[i][0]" placeholder="key (string)" /></th>
        <td><input type="text" v-model="plugins.new[i][1]" placeholder="value (JSON)" /></td>
      </tr>
      <tr>
        <td colspan="2" style="text-align: center">
          <button @click="plugins.new = [...plugins.new, ['', '']]">+</button>
        </td>
      </tr>
      <tr>
        <td colspan="2">
          <button @click="savePlugins">save</button>
          <button @click="plugins = null">close</button>
        </td>
      </tr>
    </table>

    <div  
      v-if="props.hideNewMemberForm"
      class="member"
      :class="{
        'is-admin': neko.is_admin,
      }"
      v-for="({id, profile, state}) in watchingUsers"
      :key="'session-' + id" 
    >
      <div class="topbar">
        <div class="name">
          <i v-if="neko.is_admin" class="fa fa-trash-alt" @click="memberRemove(id)" title="remove" />
          {{ profile.name }}
        </div>
        <div class="controls">
          <i
            class="fa fa-shield-alt"
            :class="{
              'state-has': profile.is_admin,
            }"
            @click="neko.is_admin && updateProfile(id, { is_admin: !profile.is_admin })"
            title="is_admin"
          />
          <i
            class="fa fa-lock-open"
            :class="{
              'state-has': profile.can_login,
            }"
            @click="neko.is_admin && updateProfile(id, { can_login: !profile.can_login })"
            title="can_login"
          />
          <i
            class="fa fa-sign-in-alt"
            :class="{
              'state-has': profile.can_connect,
              'state-is': state.is_connected,
              'state-disabled': !profile.can_login,
            }"
            @click="neko.is_admin && updateProfile(id, { can_connect: !profile.can_connect })"
            title="can_connect"
          />
          <i
            class="fa fa-desktop"
            :class="{
              'state-has': profile.can_watch,
              'state-is': state.is_watching,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_watch: !profile.can_watch })"
            title="can_watch"
          />
          <i
            class="fa fa-keyboard"
            :class="{
              'state-has': profile.can_host,
              'state-is': neko.state.control.host_id == id,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_host: !profile.can_host })"
            title="can_host"
          />
          <i
            class="fa fa-microphone"
            :class="{
              'state-has': profile.can_share_media,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_share_media: !profile.can_share_media })"
            title="can_share_media"
          />
          <i
            class="fa fa-clipboard"
            :class="{
              'state-has': profile.can_access_clipboard,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_access_clipboard: !profile.can_access_clipboard })"
            title="can_access_clipboard"
          />
          <i
            class="fa fa-mouse"
            :class="{
              'state-has': profile.sends_inactive_cursor,
              'state-is':
                profile.sends_inactive_cursor &&
                neko.state.settings.inactive_cursors &&
                neko.state.cursors.some((e:any) => e.id == id),
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="
              neko.is_admin && updateProfile(id, { sends_inactive_cursor: !profile.sends_inactive_cursor })
            "
            title="sends_inactive_cursor"
          />
          <i
            class="fa fa-mouse-pointer"
            :class="{
              'state-has': profile.can_see_inactive_cursors,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="
              neko.is_admin &&
                updateProfile(id, { can_see_inactive_cursors: !profile.can_see_inactive_cursors })
            "
            title="can_see_inactive_cursors"
          />
          <i class="fa fa-puzzle-piece" @click="neko.is_admin && showPlugins(id, profile)" title="plugins" />
        </div>
      </div>
    </div>

    <p class="title" v-if="!props.hideNewMemberForm">
      <span>Sessions</span>
    </p>

    <div 
      v-if="!props.hideNewMemberForm"
      class="member"
      :class="{
        'is-admin': neko.is_admin,
      }"
      v-for="({id, profile, state}) in sessions"
      :key="'session-' + id" 
    >
      <div class="topbar">
        <div class="name">
          <i v-if="neko.is_admin" class="fa fa-trash-alt" @click="memberRemove(id)" title="remove" />
          {{ profile.name }}
        </div>
        <div class="controls">
          <i
            class="fa fa-shield-alt"
            :class="{
              'state-has': profile.is_admin,
            }"
            @click="neko.is_admin && updateProfile(id, { is_admin: !profile.is_admin })"
            title="is_admin"
          />
          <i
            class="fa fa-lock-open"
            :class="{
              'state-has': profile.can_login,
            }"
            @click="neko.is_admin && updateProfile(id, { can_login: !profile.can_login })"
            title="can_login"
          />
          <i
            class="fa fa-sign-in-alt"
            :class="{
              'state-has': profile.can_connect,
              'state-is': state.is_connected,
              'state-disabled': !profile.can_login,
            }"
            @click="neko.is_admin && updateProfile(id, { can_connect: !profile.can_connect })"
            title="can_connect"
          />
          <i
            class="fa fa-desktop"
            :class="{
              'state-has': profile.can_watch,
              'state-is': state.is_watching,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_watch: !profile.can_watch })"
            title="can_watch"
          />
          <i
            class="fa fa-keyboard"
            :class="{
              'state-has': profile.can_host,
              'state-is': neko.state.control.host_id == id,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_host: !profile.can_host })"
            title="can_host"
          />
          <i
            class="fa fa-microphone"
            :class="{
              'state-has': profile.can_share_media,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_share_media: !profile.can_share_media })"
            title="can_share_media"
          />
          <i
            class="fa fa-clipboard"
            :class="{
              'state-has': profile.can_access_clipboard,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="neko.is_admin && updateProfile(id, { can_access_clipboard: !profile.can_access_clipboard })"
            title="can_access_clipboard"
          />
          <i
            class="fa fa-mouse"
            :class="{
              'state-has': profile.sends_inactive_cursor,
              'state-is':
                profile.sends_inactive_cursor &&
                neko.state.settings.inactive_cursors &&
                neko.state.cursors.some((e:any) => e.id == id),
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="
              neko.is_admin && updateProfile(id, { sends_inactive_cursor: !profile.sends_inactive_cursor })
            "
            title="sends_inactive_cursor"
          />
          <i
            class="fa fa-mouse-pointer"
            :class="{
              'state-has': profile.can_see_inactive_cursors,
              'state-disabled': !profile.can_login || !profile.can_connect,
            }"
            @click="
              neko.is_admin &&
                updateProfile(id, { can_see_inactive_cursors: !profile.can_see_inactive_cursors })
            "
            title="can_see_inactive_cursors"
          />
          <i class="fa fa-puzzle-piece" @click="neko.is_admin && showPlugins(id, profile)" title="plugins" />
        </div>
      </div>
    </div>

    <p class="title" v-if="!props.hideNewMemberForm">
      <span>Members</span>
      <button @click="membersLoad()">reload</button>
    </p>

    <div
      class="member"
      v-if="!props.hideNewMemberForm"
      :class="{
        'is-admin': neko.is_admin,
      }"
      v-for="member in membersWithoutSessions"
      :key="'member-' + member.id"
    >
      <div class="topbar" v-if="member.profile && member.id">
        <div class="name">
          <i v-if="neko.is_admin" class="fa fa-trash-alt" @click="memberRemove(member.id)" title="remove" />
          {{ member.profile.name }}
        </div>
        <div class="controls">
          <i
            class="fa fa-shield-alt"
            :class="{'state-has': member.profile.is_admin}"
            @click="neko.is_admin && updateProfile(member.id, { is_admin: !member.profile.is_admin })"
            title="is_admin"
          />
          <i
            class="fa fa-lock-open"
            :class="{'state-has': member.profile.can_login}"
            @click="neko.is_admin && updateProfile(member.id, { can_login: !member.profile.can_login })"
            title="can_login"
          />
          <i
            class="fa fa-sign-in-alt"
            :class="{'state-has': member.profile.can_connect, 'state-disabled': !member.profile.can_login}"
            @click="neko.is_admin && updateProfile(member.id, { can_connect: !member.profile.can_connect })"
            title="can_connect"
          />
          <i
            class="fa fa-desktop"
            :class="{'state-has': member.profile.can_watch, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { can_watch: !member.profile.can_watch })"
            title="can_watch"
          />
          <i
            class="fa fa-keyboard"
            :class="{'state-has': member.profile.can_host, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { can_host: !member.profile.can_host })"
            title="can_host"
          />
          <i
            class="fa fa-microphone"
            :class="{'state-has': member.profile.can_share_media, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { can_share_media: !member.profile.can_share_media })"
            title="can_share_media"
          />
          <i
            class="fa fa-clipboard"
            :class="{'state-has': member.profile.can_access_clipboard, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { can_access_clipboard: !member.profile.can_access_clipboard })"
            title="can_access_clipboard"
          />
          <i
            class="fa fa-mouse"
            :class="{'state-has': member.profile.sends_inactive_cursor, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { sends_inactive_cursor: !member.profile.sends_inactive_cursor })"
            title="sends_inactive_cursor"
          />
          <i
            class="fa fa-mouse-pointer"
            :class="{'state-has': member.profile.can_see_inactive_cursors, 'state-disabled': !member.profile.can_login || !member.profile.can_connect}"
            @click="neko.is_admin && updateProfile(member.id, { can_see_inactive_cursors: !member.profile.can_see_inactive_cursors })"
            title="can_see_inactive_cursors"
          />
          <i class="fa fa-puzzle-piece state-has" @click="showPlugins(member.id, member.profile)" title="plugins" />
        </div>
      </div>
    </div>


    <table class="new-member" v-if="neko.is_admin && !props.hideNewMemberForm">
    <tr>
      <td colspan="2" class="name">New Member</td>
    </tr>
    <tr>
      <th>username</th>
      <td><input type="text" v-model="newUsername" /></td>
    </tr>
    <tr>
      <th>password</th>
      <td><input type="text" v-model="newPassword" /></td>
    </tr>
    <tr>
      <td colspan="2" class="name" style="text-align: center">Profile</td>
    </tr>
    <tr>
      <th>name</th>
      <td><input type="text" v-model="newProfile.name" /></td>
    </tr>
    <tr>
      <th>is_admin</th>
      <td><input type="checkbox" v-model="newProfile.is_admin" /></td>
    </tr>
    <tr>
      <th>can_login</th>
      <td><input type="checkbox" v-model="newProfile.can_login" /></td>
    </tr>
    <tr>
      <th>can_connect</th>
      <td><input type="checkbox" v-model="newProfile.can_connect" /></td>
    </tr>
    <tr>
      <th>can_watch</th>
      <td><input type="checkbox" v-model="newProfile.can_watch" /></td>
    </tr>
    <tr>
      <th>can_host</th>
      <td><input type="checkbox" v-model="newProfile.can_host" /></td>
    </tr>
    <tr>
      <th>can_share_media</th>
      <td><input type="checkbox" v-model="newProfile.can_share_media" /></td>
    </tr>
    <tr>
      <th>can_access_clipboard</th>
      <td><input type="checkbox" v-model="newProfile.can_access_clipboard" /></td>
    </tr>
    <tr>
      <th>sends_inactive_cursor</th>
      <td><input type="checkbox" v-model="newProfile.sends_inactive_cursor" /></td>
    </tr>
    <tr>
      <th>can_see_inactive_cursors</th>
      <td><input type="checkbox" v-model="newProfile.can_see_inactive_cursors" /></td>
    </tr>
    <tr>
      <td colspan="2"><button @click="memberCreate">create</button></td>
    </tr>
  </table>
  </div>
</template>

<style lang="scss" scoped>
  @import '@/page/assets/styles/main.scss';

  .title {
    padding: 4px;
    display: flex;
    justify-content: space-between;
    align-items: center;
  }

  .members {
    display: block;
    width: 99%;
    overflow: hidden;
    font-weight: bold;
    padding-left: 1%;

    .member {
      padding: 5px;
      font-weight: bold;
      margin: 5px 5px 5px 5px;
      border: 0px solid white;
      box-sizing: border-box;

      &.is-admin .fa {
        cursor: pointer;
      }

      .topbar {
        display: flex;
        align-items: center;

        .name {
          flex: 1 1;
        }
      }

      .fa {
        padding: 5px;
        color: rgb(211, 47, 47);

        &.state-has {
          color: #fff;
        }

        &.state-is {
          color: green;
        }

        &.state-disabled {
          color: #555;
        }
      }
    }

    .new-member,
    .plugins {
      width: 100%;
      margin: 5px 0;

      .name {
        font-weight: bold;
      }

      td,
      th {
        border: 1px solid #ccc;
        padding: 4px;
        width: 50%;
      }

      th {
        text-align: right;
      }

      input[type='text'] {
        width: 100%;
        box-sizing: border-box;
      }
    }

    .plugins {
      position: absolute;
      width: auto;
      box-shadow: 0px 0px 10px 5px black;
      background: $background-tertiary;

      textarea,
      input {
        width: 100%;
        box-sizing: border-box;
      }
    }
  }
</style>

<script lang="ts" setup>
import { ref, computed, onMounted, watch, onBeforeUnmount } from 'vue'
import type Neko from '@/component/main.vue'

// TODO: get from lib ts?
import type * as ApiModels from '@/component/api/models'
import type * as StateModels from '@/component/types/state'

const props = defineProps<{
  neko: typeof Neko;
  hideNewMemberForm?: boolean;
}>();

const sessions = computed(() => props.neko.state.sessions as Record<string, StateModels.Session>)
const membersWithoutSessions = computed(() => {
  return members.value.filter(({ id }: ApiModels.MemberData) => id && !(id in sessions.value)) as ApiModels.MemberData[]
})
const watchingUsers = computed(() => {
  return Object.values(sessions.value).filter(
    (session) => session.state.is_connected || session.state.is_watching
  )
})
const members = ref<ApiModels.MemberData[]>([])
const plugins = ref<{
  id: string
  old: Array<Array<string>>
  new: Array<Array<string>>
  profile: ApiModels.MemberProfile
} | null>(null)

const newUsername = ref('')
const newPassword = ref('')
const newProfile = ref<ApiModels.MemberProfile>({})
const defProfile = ref<ApiModels.MemberProfile>({
  name: '',
  is_admin: false,
  can_login: true,
  can_connect: true,
  can_watch: true,
  can_host: true,
  can_share_media: true,
  can_access_clipboard: true,
  sends_inactive_cursor: true,
  can_see_inactive_cursors: true,
})
newProfile.value = Object.assign({}, defProfile.value)

async function memberCreate() {
  try {
    const res = await props.neko.members.membersCreate({
      username: newUsername.value.toLowerCase(), // for consistency in the file type of user creatio
      password: newPassword.value,
      profile: newProfile.value,
    })

    if (res.data) {
      members.value = [...members.value, res.data]
    }

    // clear
    newUsername.value = ''
    newPassword.value = ''
    newProfile.value = Object.assign({}, defProfile.value)
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function membersLoad(limit: number = 0) {
  const offset = 0

  try {
    const res = await props.neko.members.membersList(limit, offset)
    members.value = res.data
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function memberGetProfile(memberId: string): Promise<ApiModels.MemberProfile | undefined> {
  try {
    const res = await props.neko.members.membersGetProfile(memberId)
    return res.data
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function updateProfile(memberId: string, memberProfile: ApiModels.MemberProfile) {
  try {
    await props.neko.members.membersUpdateProfile(memberId, memberProfile)
    const newMembers = members.value.map((member) => {
      if (member.id == memberId) {
        return {
          id: memberId,
          profile: { ...member.profile, ...memberProfile },
        }
      } else {
        return member
      }
    })
    members.value = newMembers // TODO: Vue.Set
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function updatePassword(memberId: string, password: string) {
  try {
    await props.neko.members.membersUpdatePassword(memberId, { password })
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

async function memberRemove(memberId: string) {
  try {
    await props.neko.members.membersRemove(memberId)
    const newMembers = members.value.filter(({ id }) => id != memberId)
    members.value = newMembers // TODO: Vue.Set
  } catch (e: any) {
    alert(e.response ? e.response.data.message : e)
  }
}

function showPlugins(id: string, profile: ApiModels.MemberProfile) {
  const old = Object.entries(profile.plugins || {}).map(([key, val]) => [key, JSON.stringify(val, null, 2)])

  plugins.value = {
    id,
    old,
    new: old.length > 0 ? [] : [['', '']],
    profile,
  }
}

async function savePlugins() {
  if (!plugins.value) return

  let errKey = ''
  try {
    let p = {} as any
    for (let [key, val] of plugins.value.old) {
      errKey = key
      p[key] = JSON.parse(val)
    }
    for (let [key, val] of plugins.value.new) {
      errKey = key
      p[key] = JSON.parse(val)
    }

    await updateProfile(plugins.value.id, { plugins: p })
    plugins.value = null
  } catch (e: any) {
    alert(errKey + ': ' + e)
  }
}

// Method to pull the actively watching users on client init. Dunno if there's a better way to do this but this is what it took for me to get it to work.

  // Function to save user list to localStorage
  const saveUserListToCache = () => {
    if (Object.keys(sessions.value).length > 0) { 
      localStorage.setItem('cachedUserList', JSON.stringify(sessions.value));
    }
  };

  // Function to load user list from localStorage
  const loadUserListFromCache = () => {
    const cachedData = localStorage.getItem('cachedUserList');
    if (cachedData) {
      sessions.value = JSON.parse(cachedData);
    }
  };

  function clearUserListFromCache() {
    localStorage.removeItem('cachedUserList');
  }


  onMounted(() => {
    // Load from cache on mount
    if (props.neko!.isFirstMount) {
      clearUserListFromCache(); 
      props.neko!.isFirstMount = false;
    } else {
      loadUserListFromCache();
    }

    if (props.neko!.state.connected) {
      membersLoad(10); 
    }
  });

  // Save to cache before unmount and when sessions.value changes
  onBeforeUnmount(saveUserListToCache); 
  watch(sessions, saveUserListToCache, { deep: true });


props.neko.events.on('session.created', (id: string, profile: StateModels.MemberProfile, state: StateModels.SessionState) => {
    let session = { id, profile, ...profile, state, ...state} as StateModels.Session 
    sessions.value[session.id] = session
    let tempcheck = computed(() => sessions.value[session.id].state.is_connected)
    console.log(sessions.value[session.id])
    console.log(tempcheck)
    console.log(watchingUsers.value)
    setTimeout(() => {
    if (id in sessions.value && !tempcheck.value) {
      delete sessions.value[id];
    }
  }, 3000); // Adjust the delay (in milliseconds) if needed.
});

props.neko.events.on('session.deleted', (id: string) => {
  if (id in sessions.value) {
    const { [id]: deletedSession, ...rest } = sessions.value
    sessions.value = rest
  }
})

props.neko.events.on('session.updated', (id: string, state: StateModels.SessionState) => {
  if (id in sessions.value) {
    sessions.value = {
      ...sessions.value,
      [id]: { ...sessions.value[id], state: { ...sessions.value[id].state, ...state } },
    }
  }
  console.log(sessions.value[id])
  console.log(watchingUsers.value)
})


props.neko.events.on('session.profile', (id: string, profile: StateModels.MemberProfile) => {
  if (id in sessions.value) {
    sessions.value = {
      ...sessions.value,
      [id]: { ...sessions.value[id], profile: { ...sessions.value[id].profile, ...profile } },
    }
  }
})
</script>
