<script setup>
import { ref } from "vue";
import { useRouter } from "vue-router";

const router = useRouter();

const username = ref("");
const password = ref("");
const tip = ref("");
const tipColor = ref("#c62828");
// 新增：控制密码显示/隐藏的状态
const showPassword = ref(false);

const users = [
  { role: "admin", username: "admin", password: "admin" },
  { role: "user", username: "admin1", password: "admin1" }
];

function handleSubmit() {
  const matchedUser = users.find(
    (user) =>
      user.username === username.value.trim() &&
      user.password === password.value.trim()
  );

  if (!matchedUser) {
    tipColor.value = "#c62828";
    tip.value = "用户名或密码错误，请重新输入。";
    return;
  }

  localStorage.setItem("role", matchedUser.role);
  localStorage.setItem("username", matchedUser.username);

  tipColor.value = "#2e7d32";
  tip.value = "登录成功，正在进入主页...";

  window.setTimeout(() => {
    router.push("/home");
  }, 300);
}

// 新增：切换密码显示/隐藏的方法
function togglePasswordVisibility() {
  showPassword.value = !showPassword.value;
}
</script>

<template>
  <main class="panel">
    <form class="login-form" @submit.prevent="handleSubmit">
      <input
        v-model="username"
        class="field"
        type="text"
        name="username"
        placeholder="用户名"
      />
      <!-- 修改：密码输入框改为带图标的容器 -->
      <div class="password-wrapper">
        <input
          v-model="password"
          class="field password-input"
          :type="showPassword ? 'text' : 'password'"
          name="password"
          placeholder="密码"
        />
        <!-- 密码显示/隐藏切换按钮 -->
        <button 
          type="button" 
          class="password-toggle-btn"
          @click="togglePasswordVisibility"
          aria-label="切换密码显示状态"
        >
          {{ showPassword ? "🔒" : "👁️" }}
        </button>
      </div>
      <button class="btn" type="submit">登录</button>
      <p class="tip" :style="{ color: tipColor }">{{ tip }}</p>
    </form>
  </main>
</template>