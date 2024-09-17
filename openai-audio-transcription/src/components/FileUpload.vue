<template>
    <div class="mb-6">
        <label class="block text-sm font-medium text-gray-700 mb-2">上傳音檔</label>
        <div class="flex items-center justify-center w-full">
            <label for="file-upload"
                class="flex flex-col items-center justify-center w-full h-32 border-2 border-dashed border-gray-300 rounded-lg cursor-pointer bg-gray-50 hover:bg-gray-100 transition">
                <svg class="w-8 h-8 text-gray-400" fill="none" stroke="currentColor" viewBox="0 0 24 24"
                    xmlns="http://www.w3.org/2000/svg">
                    <circle cx="12" cy="12" r="9" stroke="gray" stroke-width="2" />
                    <path d="M12 16V8m0 0L8 12m4-4l4 4" stroke-linecap="round" stroke-linejoin="round" stroke-width="2"
                        stroke="gray" />
                </svg>
                <span class="mt-2 text-sm text-gray-600">點擊或拖曳檔案至此</span>
                <input id="file-upload" type="file" accept="audio/*" @change="onFileChange" class="hidden" />
            </label>
        </div>
        <div v-if="selectedFiles.length" class="mt-4">
            <p class="text-sm text-gray-600 mb-2">已選擇檔案:</p>
            <ul class="list-disc list-inside">
                <li v-for="(file, index) in selectedFiles" :key="index" class="text-gray-700">{{ file.name }}</li>
            </ul>
        </div>
    </div>
</template>

<script>
    import {
        ref
    } from 'vue';

    export default {
        name: 'FileUpload',
        emits: ['files-selected'],
        setup(props, {
            emit
        }) {
            const selectedFiles = ref([]);

            const onFileChange = (event) => {
                const files = event.target.files;
                if (files.length > 0) {
                    selectedFiles.value = Array.from(files);
                    emit('files-selected', selectedFiles.value);
                }
            };

            return {
                selectedFiles,
                onFileChange,
            };
        },
    };
</script>

<style scoped>
    /* 你可以在這裡添加或調整元件的樣式 */
</style>