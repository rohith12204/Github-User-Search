<template>
  <div class="container">
    <h1>
      <img
        src="https://github.githubassets.com/images/modules/logos_page/GitHub-Mark.png"
        alt="GitHub Logo"
        class="logo"
      />
      GitHub User Search
    </h1>

    <div class="search-box">
      <input v-model="username" placeholder="Enter GitHub username..." />
      <button @click="getUserProfile">Search</button>
    </div>

    <div v-if="error" class="error">{{ error }}</div>

    <div v-if="userProfile" class="profile-card">
      <img :src="userProfile.avatar_url" alt="Profile picture" />
      <h2>{{ userProfile.login }}</h2>
      <div class="stats">
        <span>👥 Followers: {{ userProfile.followers }}</span>
        <span>➡️ Following: {{ userProfile.following }}</span>
        <span>📂 Repos: {{ userProfile.public_repos }}</span>
      </div>
      <a :href="userProfile.html_url" target="_blank" class="profile-link">View Full Profile 🚀</a>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const username = ref('')
const userProfile = ref(null)
const error = ref(null)

const getUserProfile = async () => {
  error.value = null
  userProfile.value = null

  try {
    const response = await fetch(`https://api.github.com/users/${username.value}`)
    if (!response.ok) {
      throw new Error('User not found')
    }
    userProfile.value = await response.json()
  } catch (err) {
    console.error('Error fetching data:', err)
    error.value = 'An error occurred while fetching data'
  }
}
</script>

<style scoped>
:global(body) {
  margin: 0;
  background: #000;
  color: white;
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: flex-start;
  padding-top: 40px;
}

.container {
  width: 100%;
  max-width: 600px;
  padding: 20px;
  text-align: center;
}

h1 {
  color: white;
  margin-bottom: 20px;
  font-size: 28px;
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 12px;
}

.logo {
  width: 36px;
  height: 36px;
}

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

button {
  padding: 10px 20px;
  background: #28a745;
  color: white;
  border: none;
  border-radius: 8px;
  cursor: pointer;
  font-size: 16px;
}

button:hover {
  background: #218838;
}

.error {
  color: red;
  margin-top: 10px;
}

.profile-card {
  background: #1e1e1e;
  padding: 20px;
  border-radius: 12px;
  box-shadow: 0 6px 20px rgba(0, 0, 0, 0.5);
  text-align: center;
  margin-top: 20px;
}

.profile-card img {
  width: 120px;
  border-radius: 50%;
  margin-bottom: 10px;
}

.profile-card h2 {
  margin: 10px 0;
  color: white;
}

.stats {
  display: flex;
  justify-content: space-around;
  margin: 15px 0;
}

.stats span {
  font-size: 14px;
  color: #ddd;
}

.profile-link {
  display: inline-block;
  margin-top: 10px;
  padding: 8px 16px;
  background: #28a745;
  color: white;
  text-decoration: none;
  border-radius: 8px;
}

.profile-link:hover {
  background: #218838;
}
</style>
