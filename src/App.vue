<script setup>
// 把 Vue 的「盒子」功能引進來才能用
import { ref } from 'vue'

// 建立一個「盒子」，用來記住使用者打的 GitHub 名字
const searchName = ref('')

// 建立一個「盒子」，用來存放 GitHub 用戶的資料（頭像、名字等）
const userData = ref(null)

// 建立一個「開關/盒子」，用來記住「有沒有錯誤」（例如查無此人）
const errorMessage = ref('')

// 建立一個「開關」，用來記住「現在是不是在等待回應」
const isLoading = ref(false)

// 當按下搜尋按鈕時，執行這個函式
const searchUser = async () => {
  // 如果使用者沒輸入名字，就不要搜尋
  if (!searchName.value.trim()) return

  // 先把舊的資料清掉、把錯誤關掉、把「正在等待」的開關打開
  userData.value = null
  errorMessage.value = ''
  isLoading.value = true

  try {
    // 跟 GitHub 說「我要找這個人的資料」
    const response = await fetch(`https://api.github.com/users/${searchName.value}`)

    // 如果 GitHub 說「找不到這個人」
    if (!response.ok) {
      errorMessage.value = '查無此人'
      return
    }

    // 把找到的資料放進盒子裡
    userData.value = await response.json()
  } catch (e) {
    // 如果網路壞掉或其他問題
    errorMessage.value = '網路好像有點問題，請稍後再試'
  } finally {
    // 不管成功或失敗，「正在等待」的開關都要關掉
    isLoading.value = false
  }
}
</script>

<template>
  <div class="container">
    <!-- 標題 -->
    <h1>GitHub 偵察機</h1>

    <!-- 輸入框：讓使用者打字的地方 -->
    <div class="search-box">
      <input 
        v-model="searchName" 
        placeholder="輸入 GitHub 用戶名稱..."
        class="search-input"
        @keyup.enter="searchUser"
      />
      <button 
        class="search-button" 
        @click="searchUser"
        :disabled="isLoading"
      >
        {{ isLoading ? '搜尋中...' : '搜尋' }}
      </button>
    </div>

    <!-- ✨ 這行指令控制「錯誤訊息什麼時候出現」 -->
    <p v-if="errorMessage" class="error">{{ errorMessage }}</p>

    <!-- 當 userData 盒子裡有資料時，才顯示這個區塊 -->
    <div v-if="userData" class="user-card">
      <!-- GitHub 頭像：讓它變成圓的 -->
      <img 
        :src="userData.avatar_url" 
        :alt="userData.login + ' 的頭像'"
        class="avatar"
      />
      <h2>{{ userData.login }}</h2>
      <p v-if="userData.name" class="name">{{ userData.name }}</p>
      <p v-if="userData.bio" class="bio">{{ userData.bio }}</p>
      <div class="stats">
        <span>Repos: {{ userData.public_repos }}</span>
        <span>Followers: {{ userData.followers }}</span>
        <span>Following: {{ userData.following }}</span>
      </div>
    </div>
  </div>
</template>

<style scoped>
/* 深色背景 */
.container {
  max-width: 500px;
  margin: 50px auto;
  text-align: center;
  font-family: Arial, sans-serif;
  background-color: #1a1a2e;
  min-height: 100vh;
  padding: 40px 20px;
  box-sizing: border-box;
  color: white;
}

h1 {
  color: #e94560;
  margin-bottom: 30px;
}

.search-box {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-bottom: 30px;
}

.search-input {
  padding: 12px 15px;
  font-size: 16px;
  border: 2px solid #e94560;
  border-radius: 25px;
  width: 250px;
  background-color: #16213e;
  color: white;
  outline: none;
}

.search-input::placeholder {
  color: #888;
}

.search-input:focus {
  border-color: #0f3460;
}

.search-button {
  padding: 12px 25px;
  font-size: 16px;
  background-color: #e94560;
  color: white;
  border: none;
  border-radius: 25px;
  cursor: pointer;
  transition: background 0.3s;
}

.search-button:hover:not(:disabled) {
  background-color: #c73e54;
}

.search-button:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* 錯誤訊息 - 只有 v-if="errorMessage" 成立時才會出現 */
.error {
  color: #ff6b6b;
  font-size: 18px;
  margin: 20px 0;
}

/* 用戶卡片 */
.user-card {
  background-color: #16213e;
  border-radius: 15px;
  padding: 30px;
  margin-top: 20px;
  box-shadow: 0 4px 15px rgba(0,0,0,0.3);
}

/* 頭像變成圓的 */
.avatar {
  width: 150px;
  height: 150px;
  border-radius: 50%; /* 這個指令讓方方正正的圖片變成圓的 */
  border: 4px solid #e94560;
  margin-bottom: 15px;
}

.name {
  color: #e94560;
  font-size: 14px;
  margin-top: -10px;
}

.bio {
  color: #ccc;
  margin: 15px 0;
  line-height: 1.5;
}

.stats {
  display: flex;
  justify-content: center;
  gap: 20px;
  margin-top: 20px;
  font-size: 14px;
  color: #888;
}
</style>
