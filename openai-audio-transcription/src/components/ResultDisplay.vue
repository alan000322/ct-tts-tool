<template>
    <div class="mt-8">
        <!-- 轉錄結果 -->
        <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">轉錄結果</label>
            <textarea v-model="localText"
                class="w-full border border-gray-300 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"
                rows="10" placeholder="轉錄結果將顯示在此..."></textarea>
        </div>

        <!-- 按鈕群 -->
        <div class="flex flex-wrap gap-4 mb-6">
            <button @click="editText"
                class="bg-yellow-500 text-white px-4 py-2 rounded-md shadow hover:bg-yellow-600 transition">
                逐字稿潤稿
            </button>

            <button @click="summarizeText"
                class="bg-purple-600 text-white px-4 py-2 rounded-md shadow hover:bg-purple-700 transition">
                逐字稿摘要
            </button>

            <button @click="translateText"
                class="bg-blue-600 text-white px-4 py-2 rounded-md shadow hover:bg-blue-700 transition">
                翻譯成中文
            </button>
            
            
        </div>

        <!-- 潤稿依據 -->
        <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">潤稿依據</label>
            <input type="text" v-model="editingBasis"
                class="w-full border border-gray-300 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-blue-500"
                placeholder="請輸入潤稿依據...：名詞1 名詞2 名詞3 依據1 依據2 依據3 ..." />
        </div>

        <!-- 結果顯示區 -->
        <div class="mb-6">
            <label class="block text-sm font-medium text-gray-700 mb-2">結果</label>
            <textarea v-model="actionContent"
                class="w-full border border-gray-300 rounded-md p-3 focus:outline-none focus:ring-2 focus:ring-blue-500 resize-none"
                rows="40" placeholder="操作結果將顯示在此..."></textarea>
        </div>
    </div>
</template>

<script>
    import {
        ref,
        watch
    } from 'vue';

    export default {
        name: 'ResultDisplay',
        props: {
            text: {
                type: String,
                required: true,
            },
            actionResult: {
                type: String,
                default: '',
            },
        },
        emits: ['action'],
        setup(props, {
            emit
        }) {
            const localText = ref(props.text);
            const actionContent = ref(props.actionResult);
            const editingBasis = ref('');

            // 當 actionResult 更新時，更新 actionContent
            watch(
                () => props.actionResult,
                (newVal) => {
                    actionContent.value = newVal;
                }
            );

            // 翻譯成英文
            const translateText = () => {
                //emit('action', 'translate', localText.value);
                emit('action', 'translate', {
                    text: localText.value,
                    basis: editingBasis.value
                });
            };

            // 逐字稿潤稿，包含潤稿依據
            const editText = () => {
                emit('action', 'edit', {
                    text: localText.value,
                    basis: editingBasis.value
                });
            };

            // 逐字稿摘要
            const summarizeText = () => {
//                emit('action', 'summarize', localText.value);
                emit('action', 'summarize', {
                    text: localText.value,
                    basis: editingBasis.value
                });
            };

            return {
                localText,
                actionContent,
                editingBasis,
                translateText,
                editText,
                summarizeText,
                // 已移除複製按鈕
            };
        },
    };
</script>

<style scoped>
    /* 你可以在這裡添加或調整元件的樣式 */
    textarea {
        transition: border-color 0.2s, box-shadow 0.2s;
    }
</style>