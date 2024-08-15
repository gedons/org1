<template>
  <div v-if="isVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white w-full max-w-4xl p-6 rounded-lg shadow-lg relative">
      <h3 class="text-lg font-semibold mb-4">Upload File</h3>

      <div 
        @drop.prevent="onDrop" 
        @dragover.prevent="onDragOver" 
        @click="triggerFileInput" 
        class="border-dashed border-2 border-gray-300 p-6 mb-4 flex items-center justify-center cursor-pointer"
        :class="{ 'bg-gray-100': !selectedFile }"
      >
        <input ref="fileInput" type="file" @change="onFileSelected" class="hidden" />
        <p v-if="!selectedFile">Drag and drop a file here or click to select</p>
        <p v-if="selectedFile">{{ selectedFile.name }}</p>
      </div>

      <div class="flex justify-end">
        <button 
          @click="uploadFile" 
          class="bg-yellow-300 text-black font-semibold py-2 px-4 rounded-md mr-2" 
          :disabled="isLoading"
        >
          <span v-if="isLoading">Uploading...</span>
          <span v-else>Upload</span>
        </button>
        <button @click="closeModal" class="bg-gray-300 text-black py-2 px-4 rounded-md">Cancel</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  props: {
    isVisible: Boolean,
    currentFolderPath: String,
    currentFolderId: String
  },
  data() {
    return {
      selectedFile: null,
      isLoading: false
    };
  },
  methods: {
    closeModal() {
      this.$emit('close');
    },

    onFileSelected(event) {
      this.selectedFile = event.target.files[0];
    },

    onDragOver(event) {
      event.preventDefault();
    },

    onDrop(event) {
      this.selectedFile = event.dataTransfer.files[0];
    },

    triggerFileInput() {
      this.$refs.fileInput.click();
    },

    async uploadFile() {
      if (!this.selectedFile) return;

      this.isLoading = true;
      const formData = new FormData();
      formData.append('file', this.selectedFile);
      formData.append('folderPath', this.currentFolderPath || '');

      // Append folderId only if it's not null
      if (this.currentFolderId) {
        formData.append('folderId', this.currentFolderId);
      }

      try {
        const token = localStorage.getItem('token');
        const response = await fetch('https://fileorganise.onrender.com/api/files/upload', {
          method: 'POST',
          body: formData,
          headers: {
            Authorization: `Bearer ${token}`,
          },
        });

        const data = await response.json();
        this.$emit('fileUploaded', data.file);
        this.closeModal();
      } catch (error) {
        console.error('Failed to upload file:', error.message);
      } finally {
        this.isLoading = false;
      }
    }
  }
};
</script>

<style scoped>
.loader {
  border: 2px solid rgba(255, 255, 255, 0.3);
  border-radius: 50%;
  border-top: 2px solid #fff;
  width: 20px;
  height: 20px;
  animation: spin 1s linear infinite;
  display: inline-block;
  margin-right: 10px;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}
</style>
