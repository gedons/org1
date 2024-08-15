<template>
  <div v-if="isVisible" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
    <div class="bg-white w-full max-w-4xl p-6 rounded-lg shadow-lg">
      <h3 class="text-lg font-semibold mb-4">Create Folder</h3>
      <input v-model="folderName" type="text" placeholder="Folder Name" class="border w-full p-2 mb-4" />
      <div class="flex justify-end">
        <button 
          @click="createFolder" 
          class="bg-yellow-300 text-black font-semibold py-2 px-4 rounded-md mr-2" 
          :disabled="isLoading"
        >
          <span v-if="isLoading">Creating...</span>
          <span v-else>Create</span>
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
    currentFolderId: String
  },
  data() {
    return {
      folderName: '',
      isLoading: false
    };
  },
  methods: {
    closeModal() {
      this.$emit('close');
    },

    async createFolder() {
      this.isLoading = true;
      try {
        const token = localStorage.getItem('token');
        const response = await fetch('https://fileorganise.onrender.com/api/folders', {
          method: 'POST',
          headers: {
            'Content-Type': 'application/json',
            Authorization: `Bearer ${token}`
          },
          body: JSON.stringify({
            folderName: this.folderName,
            parentFolderId: this.currentFolderId || null
          }),
        });

        const data = await response.json();
        this.$emit('folderCreated', data.folder);
        this.closeModal();
      } catch (error) {
        console.error('Failed to create folder:', error.message);
      } finally {
        this.isLoading = false;
      }
    }
  }
};
</script>
