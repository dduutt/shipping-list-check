<script setup>
import { onMounted, ref } from "vue";
import { open } from '@tauri-apps/plugin-dialog';
import { desktopDir } from '@tauri-apps/api/path';
import { Command } from '@tauri-apps/plugin-shell';

const filePath = ref("");
const checkResult = ref([]);
const checkList = ref("");
const msg = ref("");


onMounted(() => {
  filePath.value = localStorage.getItem('filePath');
  checkList.value = localStorage.getItem('checkList') || "图号,规格结构";
})



async function selectFile() {
  filePath.value = await open({
    multiple: false,
    directory: false,
    defaultPath: await desktopDir(),
    filters: [
      {
        name: 'Excel Files',
        extensions: ['xlsx'],
      },
    ],
  });
}

async function check() {
  msg.value = "";
  checkResult.value = [];
  if (!filePath.value) {
    alert('请先选择文件');
    return;
  }
  // 保存文件路径及核验清单
  localStorage.setItem('filePath', filePath.value);
  localStorage.setItem('checkList', checkList.value);
  const command = Command.sidecar('sidecar/shipping-check', ['-f', filePath.value, "-h", checkList.value]);
  const output = await command.execute();
  const stdout = output.stdout;
  if (output.code == 0) {
    checkResult.value = JSON.parse(output.stdout);
    if (checkResult.value.length == 0) {
      msg.value = "核验通过";
    }
    return
  }
  msg.value = stdout;
}

</script>


<template>
  <main class="container">
    <h1>发货清单核验程序</h1>

    <div class="row">
      <button @click="selectFile">选择文件</button>
    </div>
    <p>{{ filePath }}</p>

    <div class="row">
      <input id="greet-input" v-model="checkList" placeholder="Enter a name..." />
      <button @click="check">开始核验</button>
    </div>
    <p>{{ msg }}</p>
    <div class="row" v-for="item in checkResult">
      {{ item }}
    </div>
  </main>
</template>


<style>
:root {
  font-family: Inter, Avenir, Helvetica, Arial, sans-serif;
  font-size: 16px;
  line-height: 24px;
  font-weight: 400;

  color: #0f0f0f;
  background-color: #f6f6f6;

  font-synthesis: none;
  text-rendering: optimizeLegibility;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  -webkit-text-size-adjust: 100%;
}

.container {
  margin: 0;
  padding-top: 10vh;
  display: flex;
  flex-direction: column;
  justify-content: center;
  text-align: center;
}

.row {
  display: flex;
  justify-content: center;
}

a {
  font-weight: 500;
  color: #646cff;
  text-decoration: inherit;
}


h1 {
  text-align: center;
}

input,
button {
  border-radius: 8px;
  border: 1px solid transparent;
  padding: 0.6em 1.2em;
  font-size: 1em;
  font-weight: 500;
  font-family: inherit;
  color: #0f0f0f;
  background-color: #ffffff;
  transition: border-color 0.25s;
  box-shadow: 0 2px 2px rgba(0, 0, 0, 0.2);
}

button {
  cursor: pointer;
}

button:hover {
  border-color: #396cd8;
}

button:active {
  border-color: #396cd8;
  background-color: #e8e8e8;
}

input,
button {
  outline: none;
}

#greet-input {
  margin-right: 5px;
}

@media (prefers-color-scheme: dark) {
  :root {
    color: #f6f6f6;
    background-color: #2f2f2f;
  }

  a:hover {
    color: #24c8db;
  }

  input,
  button {
    color: #ffffff;
    background-color: #0f0f0f98;
  }

  button:active {
    background-color: #0f0f0f69;
  }
}
</style>