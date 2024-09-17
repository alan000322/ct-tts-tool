<template>
  <div class="min-h-screen bg-gray-100 flex items-center justify-center p-4">
    <div class="bg-white shadow-lg rounded-lg p-8 w-full max-w-4xl">
      <h1 class="text-3xl font-bold text-center mb-6 text-blue-600">CT Speech2Text</h1>

      <!-- API Key 輸入區 -->
      <div class="mb-6">
        <label for="apiKey" class="block text-sm font-medium text-gray-700 mb-1">OpenAI API Key</label>
        <input type="password" id="apiKey" v-model="apiKey"
          class="w-full border border-gray-300 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-blue-500"
          placeholder="Enter your OpenAI API Key" />
      </div>

      <!-- 音檔上傳區 -->
      <FileUpload @files-selected="handleFilesSelected" />

      <!-- 傳送按鈕 -->
      <div class="flex justify-center mt-6">
        <button @click="handleSend" :disabled="!apiKey || !files.length || loading"
          class="bg-blue-600 text-white px-6 py-3 rounded-md shadow hover:bg-blue-700 disabled:bg-gray-400 transition duration-200">
          <span v-if="!loading">傳送</span>
          <span v-else>傳送中...</span>
        </button>
      </div>

      <!-- Loading 動畫 -->
      <LoadingSpinner v-if="loading" />

      <!-- 結果顯示區 -->
      <ResultDisplay v-if="result" :text="result.transcript" @action="handleAction" :actionResult="actionResult" />
    </div>
  </div>
</template>

<script>
  import {
    ref
  } from 'vue';
  import axios from 'axios';
  import FileUpload from './components/FileUpload.vue';
  import ResultDisplay from './components/ResultDisplay.vue';
  import LoadingSpinner from './components/LoadingSpinner.vue';

  export default {
    name: 'App',
    components: {
      FileUpload,
      ResultDisplay,
      LoadingSpinner,
    },
    setup() {
      const apiKey = ref('');
      const files = ref([]);
      const loading = ref(false);
      const result = ref(null);
      const actionResult = ref(null);

      // 處理選擇的檔案
      const handleFilesSelected = (selectedFiles) => {
        files.value = selectedFiles;
      };

      // 處理傳送按鈕點擊
      const handleSend = async () => {
        if (!apiKey.value || files.value.length === 0) {
          alert('請輸入 API Key 並選擇至少一個檔案。');
          return;
        }

        loading.value = true;
        actionResult.value = null;

        try {
          // 使用 OpenAI Whisper API 進行轉錄
          const formData = new FormData();
          formData.append('file', files.value[0]); // 只處理第一個檔案
          formData.append('model', 'whisper-1');
          formData.append('response_format', 'text'); // 指定回傳格式

          const response = await axios.post('https://api.openai.com/v1/audio/transcriptions',
            formData, {
              headers: {
                'Content-Type': 'multipart/form-data',
                Authorization: `Bearer ${apiKey.value}`, // 確保這裡取得最新的 apiKey 值
              },
            });

          result.value = {
            transcript: response.data,//.text,
            // transcript: response.data.text,
          };
        } catch (error) {
          console.error(error);
          alert('轉錄失敗，請檢查 API Key 或檔案格式。');
        } finally {
          loading.value = false;
        }
      };

      // 處理 ResultDisplay 的動作按鈕
      const handleAction = async (action, payload) => {
        if (!apiKey.value) {
          alert('請輸入 API Key。');
          return;
        }

        if (action === 'translate') {
          const {
            text,
            basis
          } = payload;

          if (!basis) {
            alert('請在「潤稿依據」欄位輸入潤稿的依據。');
            return;
          }

          loading.value = true;
          try {
            const prompt = `將文稿結果，翻譯成英文，請注意專有名詞（人名、地名）請翻譯成英文實體名稱後，括號中文名稱。另外，使用者提供一些潤稿依據，也請遵守:\n\n${basis}`;
            const completionResponse = await axios.post(
              'https://api.openai.com/v1/chat/completions', {
                model: 'gpt-4o-2024-08-06',
                messages: [{
                    "role": "system",
                    "content": prompt
                  },
                  {
                    "role": "user",
                    "content": text
                  }
                ],
                max_tokens: 16000,
                temperature: 1.0,
              }, {
                headers: {
                  'Content-Type': 'application/json',
                  Authorization: `Bearer ${apiKey.value}`, // 確保這裡取得最新的 apiKey 值
                },
              }
            );

            actionResult.value = completionResponse.data.choices[0].message.content.trim();
          } catch (error) {
            console.error(error);
            alert('翻譯失敗，請檢查 API Key 或稍後再試。');
          } finally {
            loading.value = false;
          }
        } else if (action === 'edit') {
          const {
            text,
            basis
          } = payload;
          if (!basis) {
            alert('請在「潤稿依據」欄位輸入潤稿的依據。');
            return;
          }

          loading.value = true;
          try {
            const prompt =
              `你 重要的改稿編輯，當使用者給你任何文件，請不更改任何文件內容，僅更改使用者所提供給你的一些名詞資訊，加以替換、修改：\n${basis}\n，且若為半形空格「 」，請根據文意新增一些繁體中文的標點符號（「」，。、！？），此外適度的進行分段，其餘不做任何更正`;
            const completionResponse = await axios.post(
              'https://api.openai.com/v1/chat/completions', {
                model: 'gpt-4o-2024-08-06',
                messages: [{
                    "role": "system",
                    "content": prompt
                  },
                  {
                    "role": "user",
                    "content": text
                  }
                ],
                max_tokens: 16000,
                temperature: 1.0,
              }, {
                headers: {
                  'Content-Type': 'application/json',
                  Authorization: `Bearer ${apiKey.value}`, // 確保這裡取得最新的 apiKey 值
                },
              }
            );

            actionResult.value = completionResponse.data.choices[0].message.content.trim();
          } catch (error) {
            console.error(error);
            alert('潤稿失敗，請檢查 API Key 或稍後再試。');
          } finally {
            loading.value = false;
          }
        } else if (action === 'summarize') {
          const {
            text,
            basis
          } = payload;
          if (!basis) {
            alert('請在「潤稿依據」欄位輸入潤稿的依據。');
            return;
          }
          loading.value = true;
          try {
            const prompt = `將使用者給你的逐字稿加以摘要，最終以正式文稿（像是新聞一樣的）輸出。其中也須符合使用者給訂的以下內容（可能包含專有名詞處理、錯別字、實體命名等）:\n\n${basis}`;
            const completionResponse = await axios.post(
              'https://api.openai.com/v1/chat/completions', {
                model: 'gpt-4o-2024-08-06',
                messages: [{
                    "role": "system",
                    "content": prompt
                  },
                  {
                    "role": "user",
                    "content": text
                  }
                ],
                max_tokens: 16000,
                temperature: 1.0,
              }, {
                headers: {
                  'Content-Type': 'application/json',
                  Authorization: `Bearer ${apiKey.value}`, // 確保這裡取得最新的 apiKey 值
                },
              }
            );

            actionResult.value = completionResponse.data.choices[0].message.content.trim();
          } catch (error) {
            console.error(error);
            alert('摘要失敗，請檢查 API Key 或稍後再試。');
          } finally {
            loading.value = false;
          }
        }
      };

      return {
        apiKey,
        files,
        loading,
        result,
        actionResult,
        handleFilesSelected,
        handleSend,
        handleAction,
      };
    },
  };
</script>

<style>
  /* 你可以在這裡添加全局樣式 */
</style>