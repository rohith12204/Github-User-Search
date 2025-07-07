<template>
  <div class="container">
    <!-- History container on left corner -->
    <div class="history-box">
      <h3>🔎 History</h3>
      <ul>
        <li v-for="(user, index) in history" :key="index" @click="loadFromHistory(user)">
          {{ user }}
        </li>
      </ul>
    </div>

    <!-- Header section with GitHub logo + toggle -->
    <div class="top-bar">
      <h1>
        <img
          src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png"
          alt="GitHub Logo"
          class="logo"
        />
        GitHub User Search
      </h1>

      <button @click="toggleTheme" class="theme-toggle">
        {{ theme === 'dark' ? '🌞 Light Mode' : '🌙 Dark Mode' }}
      </button>
    </div>

    <div class="search-box">
      <input
        v-model="username"
        @keyup.enter="getUserProfile"
        placeholder="Enter GitHub username..."
      />
      <button @click="getUserProfile">Search</button>
    </div>

    <div v-if="loading" class="loading-spinner">🔄 Loading...</div>
    <div v-if="error" class="error">{{ error }}</div>

    <transition name="fade">
      <div v-if="userProfile" class="profile-card">
        <img :src="userProfile.avatar_url" alt="Profile picture" class="profile-img" />
        <h2>{{ userProfile.login }}</h2>
        <p v-if="userProfile.bio">{{ userProfile.bio }}</p>
        <p v-if="userProfile.company">🏢 {{ userProfile.company }}</p>
        <p v-if="userProfile.location">📍 {{ userProfile.location }}</p>
        <p v-if="userProfile.blog">
          🔗
          <a
            :href="formatBlogUrl(userProfile.blog)"
            target="_blank"
            class="profile-link inline-link"
            >Blog</a
          >
        </p>
        <p>📅 Joined: {{ formatDate(userProfile.created_at) }}</p>

        <div class="stats">
          <span>👥 Followers: {{ userProfile.followers }}</span>
          <span>➡️ Following: {{ userProfile.following }}</span>
          <span>📂 Repos: {{ userProfile.public_repos }}</span>
        </div>

        <a :href="userProfile.html_url" target="_blank" class="profile-link"
          >View Full Profile 🚀</a
        >
      </div>
    </transition>
  </div>
</template>

<script setup>
import { ref, watch, onMounted } from 'vue'

const username = ref('')
const userProfile = ref(null)
const error = ref(null)
const loading = ref(false)
const theme = ref('dark')
const history = ref([])

const toggleTheme = () => {
  theme.value = theme.value === 'dark' ? 'light' : 'dark'
}

const getUserProfile = async () => {
  if (!username.value.trim()) return

  error.value = null
  userProfile.value = null
  loading.value = true

  try {
    const response = await fetch(`https://api.github.com/users/${username.value}`)
    if (!response.ok) {
      if (response.status === 403) {
        throw new Error('Rate limit exceeded. Try again later.')
      }
      throw new Error('User not found')
    }
    userProfile.value = await response.json()

    if (!history.value.includes(username.value)) {
      history.value.unshift(username.value)
    }
  } catch (err) {
    console.error('Error fetching data:', err)
    error.value = err.message
  } finally {
    loading.value = false
  }
}

const loadFromHistory = (name) => {
  username.value = name
  getUserProfile()
}

const formatBlogUrl = (url) => {
  return url.startsWith('http') ? url : `https://${url}`
}
const formatDate = (date) => {
  return new Date(date).toLocaleDateString()
}

onMounted(() => {
  document.body.classList.add(theme.value)
})
watch(theme, (newTheme, oldTheme) => {
  document.body.classList.remove(oldTheme)
  document.body.classList.add(newTheme)
})
</script>

<style scoped>
:global(body) {
  margin: 0;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  min-height: 100vh;
  padding-top: 40px;
  transition:
    background 0.3s,
    color 0.3s;
}

:global(.dark) {
  background: #000;
  color: white;
}
:global(.light) {
  background: #f5f5f5;
  color: #333;
}

.container {
  width: 90%;
  max-width: 600px;
  margin: 0 auto;
  text-align: center;
}

/* HEADER BAR */
.top-bar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  flex-wrap: wrap;
  margin-bottom: 20px;
  margin-left: 150px;
}

h1 {
  font-size: 28px;
  display: flex;
  align-items: center;
  gap: 12px;
}
.logo {
  width: 36px;
  height: 36px;
  transition: transform 0.3s;
}
.logo:hover {
  transform: rotate(15deg) scale(1.2);
}

.theme-toggle {
  background: transparent;
  border: 2px solid currentColor;
  padding: 6px 12px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
}
.theme-toggle:hover {
  background: currentColor;
  color: #fff;
}

/* SEARCH BAR */
.search-box {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 20px;
}
input {
  flex: 1;
  padding: 10px;
  border: 2px solid #555;
  background: #222;
  color: white;
  border-radius: 8px;
  outline: none;
  font-size: 16px;
}
:global(.light) input {
  background: #fff;
  color: #333;
  border: 2px solid #ccc;
}

button {
  padding: 10px 20px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
  transition: transform 0.2s;
}
button:hover {
  background: #218838;
  transform: scale(1.05);
}

/* LOADING & ERROR */
.loading-spinner {
  margin-top: 20px;
  font-size: 18px;
  color: #28a745;
}
.error {
  color: red;
  margin-top: 10px;
}

/* PROFILE CARD */
.profile-card {
  background: #1e1e1e;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.5);
  text-align: center;
  margin-top: 20px;
  transition: all 0.3s ease;
}
:global(.light) .profile-card {
  background: #fff;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}
.profile-card:hover {
  transform: translateY(-4px);
}
.profile-card img.profile-img {
  width: 120px;
  border-radius: 50%;
  margin-bottom: 10px;
  transition: transform 0.3s;
}
.profile-card img.profile-img:hover {
  transform: scale(1.1) rotate(5deg);
}
.profile-card h2,
.profile-card p {
  margin: 4px 0;
}
:global(.light) .profile-card h2,
:global(.light) .profile-card p {
  color: #333;
}

/* STATS */
.stats {
  display: flex;
  justify-content: space-around;
  margin: 15px 0;
  flex-wrap: wrap;
}
.stats span {
  font-size: 14px;
}

/* LINKS */
.profile-link {
  display: inline-block;
  margin-top: 10px;
  padding: 8px 16px;
  background: #28a745;
  color: white;
  text-decoration: none;
  border-radius: 8px;
  transition: background 0.2s;
}
.profile-link:hover {
  background: #218838;
}
.inline-link {
  padding: 2px 6px;
  background: #007acc;
  border-radius: 6px;
  color: white;
  text-decoration: none;
  font-size: 13px;
}
.inline-link:hover {
  background: #005fa3;
}

/* HISTORY BOX IN LEFT CORNER */
.history-box {
  position: absolute;
  left: 20px;
  top: 133px;
  width: 220px;
  background: #1e1e1e;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.5);
  transition: all 0.3s ease;
}
:global(.light) .history-box {
  background: #fff;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.1);
}
.history-box:hover {
  transform: translateY(-4px);
}
.history-box h3 {
  font-size: 18px;
  margin-bottom: 10px;
}
.history-box ul {
  list-style: none;
  padding: 0;
}
.history-box li {
  padding: 6px;
  border-bottom: 1px solid #444;
  cursor: pointer;
  transition: background 0.2s;
}
.history-box li:hover {
  background: #333;
}
:global(.light) .history-box li:hover {
  background: #ddd;
}

/* FADE */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.5s;
}
.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}
</style>
