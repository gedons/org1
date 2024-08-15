<template>
    <div class="min-h-screen flex flex-col">
      <!-- Navbar and Header -->
      <nav class="bg-blue-600 p-4 text-white flex justify-between items-center">
        <h1 class="text-2xl font-semibold">File Organize</h1>
        <div class="relative">
          <button @click="toggleDropdown" class="focus:outline-none">
            <img src="../assets/profile.svg" alt="Profile" class="w-8 h-6 rounded-full" />
          </button>
          <div v-if="dropdownVisible" class="absolute right-0 mt-2 w-48 bg-white rounded-md shadow-lg z-10">
            <div class="py-2">
              <button @click="logout" class="block px-4 py-2 text-gray-800 hover:bg-gray-200 w-full text-left">Logout</button>
            </div>
          </div>
        </div>
        </nav>
  
      <!-- Main Content Area -->
      <div class="flex-1 p-6 flex flex-col">
        <div class="flex justify-between items-center mb-4">
          <h2 class="text-xl font-semibold">My Files & Folders</h2>
          <div v-if="!currentFolder" class="flex flex-col sm:flex-row sm:justify-start sm:space-x-2">
            <button @click="showCreateFolder" class="bg-blue-600 font-semibold text-white py-2 px-4 rounded-md mb-2 sm:mb-0">Create Folder</button>
            <button @click="showUploadFile" class="bg-yellow-300 font-semibold text-black py-2 px-4 rounded-md">Upload File</button>
          </div>
          <div v-else class="flex flex-col sm:flex-row sm:justify-start sm:space-x-2">
            <button @click="goBack" class="bg-yellow-300 text-black py-2 px-4 font-semibold rounded-md mb-2 sm:mb-0">Back</button>
            <!-- <button @click="showCreateFolder" class="bg-blue-600 font-semibold text-white py-2 px-4 rounded-md mb-2 sm:mb-0">Create Folder</button>
            <button @click="showUploadFile" class="bg-yellow-300 font-semibold text-black py-2 px-4 rounded-md">Upload File</button> -->
          </div>
        </div>

        <!-- Modals -->
        <CreateFolderModal 
          :isVisible="isCreateFolderModalVisible" 
          :currentFolderId="currentFolder ? currentFolder._id : null"
          @close="isCreateFolderModalVisible = false"
          @folderCreated="addFolder"
        />
        
        <UploadFileModal 
          :isVisible="isUploadFileModalVisible"
          :currentFolderPath="currentFolder ? currentFolder.s3Key : ''"
          :currentFolderId="currentFolder ? currentFolder._id : null"
          @close="isUploadFileModalVisible = false"
          @fileUploaded="addFile"
        />

        <div class="mb-4 flex justify-end">
          <button @click="toggleView" class="py-2 px-2 rounded-md">
            <img src="../assets/list.svg" alt="Folder" class="w-8 h-8 mt-4" v-if="isGridView" />
            <img src="../assets/grid.svg" alt="File" class="w-8 h-8 mt-4" v-if="!isGridView" />
            <p class="font-semibold text-sm">Toggle View</p>
          </button>
        </div>

        <h2 class="text-xl font-semibold mb-2">All Files</h2>

        <!-- Progress Bar -->
        <div v-if="isLoading" class=" mb-2 w-full h-1 bg-blue-500 z-50">
          <div class="h-full bg-green-500" :style="{ width: progress + '%' }"></div>
        </div>
        
        <!-- Spinner -->
        <div v-if="isFetching" class="fixed inset-0 flex items-center justify-center bg-white bg-opacity-75 z-50">
          <div class="loader"></div>
        </div>

        <div v-else :class="isGridView ? 'grid grid-cols-1 sm:grid-cols-2 lg:grid-cols-3 gap-4' : 'ml-6 flex flex-col w-full'">
          
          <!-- Display Folders -->
          <div v-if="folders.length > 0 && !currentFolder" v-for="folder in folders" :key="folder._id" 
              class="mb-4 cursor-pointer" 
              :class="isGridView ? 'border p-4 rounded-md' : 'border-b w-full'"
              @click="openFolder(folder)" 
              @dragover.prevent 
              @drop="handleDrop($event, folder)">
            <div class="folder-icon">
              <img src="../assets/folder.svg" alt="Folder" class="w-12 h-12" />
            </div>
            <div class="folder-name font-semibold">{{ folder.folderName }}</div>
            <button @click.stop="deleteFolder(folder._id)" class="text-red-500 font-semibold text-sm mt-2">Delete</button>
          </div>

          <!-- Display Files in Current Folder -->
          <div v-if="currentFolder && currentFolder.files.length > 0" v-for="file in currentFolder.files" :key="file._id"
              class="file-card mb-4" 
              :class="isGridView ? 'border p-4 rounded-md' : 'border-b w-full'"
              draggable="true" 
              @dragstart="handleDragStart($event, file)">
            <div v-if="isImage(file.fileType)" class="file-preview">
              <img :src="file.s3Url" :alt="file.fileName" class="w-24 h-32 object-cover" />
            </div>
            <div v-else-if="isDocument(file.fileType)" class="file-preview">
              <img src="../assets/document.svg" alt="Document" class="w-12 h-12" />
            </div>
            <div v-else-if="isSpreadsheet(file.fileType)" class="file-preview">
              <img src="../assets/excel.svg" alt="Spreadsheet" class="w-12 h-12" />
            </div>
            <div v-else-if="isPDF(file.fileType)" class="file-preview">
              <img src="../assets/pdf.svg" alt="PDF" class="w-12 h-12" />
            </div>
            <div v-else-if="isCSV(file.fileType)" class="file-preview">
              <img src="../assets/excel.svg" alt="CSV" class="w-12 h-12" />
            </div>
            <div v-else-if="isTXT(file.fileType)" class="file-preview">
              <img src="../assets/file.svg" alt="TXT" class="w-12 h-12" />
            </div>
            <div class="file-name font-semibold">
              <a :href="file.s3Url" target="_blank">{{ file.fileName }}</a>
            </div>
            <button @click.stop="deleteFile(file._id)" class="text-red-500 font-semibold text-sm mt-2">Delete</button>
          </div>

          <!-- Display Files in Root -->
          <div v-if="!currentFolder && files.length > 0" v-for="file in files" :key="file._id"
              class="file-card mb-4" 
              :class="isGridView ? 'border p-4 rounded-md' : 'border-b w-full'"
              draggable="true" 
              @dragstart="handleDragStart($event, file)">
            <div v-if="isImage(file.fileType)" class="file-preview">
              <img :src="file.s3Url" :alt="file.fileName" class="w-24 h-32 object-cover" />
            </div>
            <div v-else-if="isDocument(file.fileType)" class="file-preview">
              <img src="../assets/document.svg" alt="Document" class="w-12 h-12" />
            </div>
            <div v-else-if="isSpreadsheet(file.fileType)" class="file-preview">
              <img src="../assets/excel.svg" alt="Spreadsheet" class="w-12 h-12" />
            </div>
            <div v-else-if="isPDF(file.fileType)" class="file-preview">
              <img src="../assets/pdf.svg" alt="PDF" class="w-12 h-12" />
            </div>
            <div v-else-if="isCSV(file.fileType)" class="file-preview">
              <img src="../assets/excel.svg" alt="CSV" class="w-12 h-12" />
            </div>
            <div v-else-if="isTXT(file.fileType)" class="file-preview">
              <img src="../assets/file.svg" alt="TXT" class="w-12 h-12" />
            </div>
            <div class="file-name font-semibold">
              <a :href="file.s3Url" target="_blank">{{ file.fileName }}</a>
            </div>
            <button @click.stop="deleteFile(file._id)" class="text-red-500 font-semibold text-sm mt-2">Delete</button>
          </div>

          <!-- No Files or Folders Available Message -->
          <div v-if="(folders.length === 0 && !currentFolder && files.length === 0) || (currentFolder && currentFolder.files.length === 0)"
              class="text-left text-gray-500 font-semibold mt-10">
            <p>No files or folders available</p>
          </div>
        </div>
      </div>

      </div>
    </template>
  
    <script>
    import CreateFolderModal from './CreateFolderModal.vue';
    import UploadFileModal from './UploadFileModal.vue';
    
    export default {
      components: {
        CreateFolderModal,
        UploadFileModal
      },
      data() {
        return {
          files: [],
          folders: [],
          isCreateFolderModalVisible: false,
          isUploadFileModalVisible: false,
          currentFolder: null,
          isGridView: true, 
          dropdownVisible: false,
          draggedFile: null, 
          isFetching : false,
          isLoading: false,
          progress: 0,
        };
      },
      methods: {
        startProgress() {
          this.progress = 0;
          this.isLoading = true;

          const interval = setInterval(() => {
            if (this.progress < 90) {
              this.progress += 5;  
            }
          }, 100);  

          return interval;  
        },

        showCreateFolder() {
        this.isCreateFolderModalVisible = true;
        },

        showUploadFile() {
          this.isUploadFileModalVisible = true;
        },

        addFolder(folder) {
          if (this.currentFolder) {
            this.currentFolder.folders.push(folder);
          } else {
            this.folders.push(folder);
          }
        },

        addFile(file) {
          if (this.currentFolder) {
            this.currentFolder.files.push(file);
          } else {
            this.files.push(file);
          }
        },

        async fetchFolders() {
          this.isFetching = true;
          try {
            const response = await fetch('https://fileorganise.onrender.com/api/folders', {
              headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`,
              },
            });
            const data = await response.json();
            this.folders = data;
          } catch (error) {
            console.error('Error fetching folders:', error);
          }finally {
            this.isFetching = false;
          }
        },
    
        async fetchFiles() {
          this.isFetching = true;
          try {
            const response = await fetch('https://fileorganise.onrender.com/api/files', {
              headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`,
              },
            });
            const data = await response.json();
            this.files = data.filter(file => !file.folder);
            this.folders.forEach(folder => {
              folder.files = data.filter(file => file.folder === folder._id) || [];
            });
          } catch (error) {
            console.error('Error fetching files:', error);
          }finally {
            this.isFetching = false;
          }
        },

        async deleteFile(fileId) {
            if (confirm('Are you sure you want to delete this file?')) {
             const interval = this.startProgress();
            try {
                const response = await fetch(`https://fileorganise.onrender.com/api/files/${fileId}`, {
                method: 'DELETE',
                headers: {
                    'Authorization': `Bearer ${localStorage.getItem('token')}`,
                },
                });

                if (response.ok) {
                this.files = this.files.filter(file => file._id !== fileId);
                if (this.currentFolder) {
                    this.currentFolder.files = this.currentFolder.files.filter(file => file._id !== fileId);
                }
                this.progress = 100; 
                } else {
                console.error('Error deleting file:', await response.text());
                }
              } catch (error) {
                  console.error('Error deleting file:', error);
              }finally {
                clearInterval(interval);
                setTimeout(() => {
                  this.isLoading = false;
                  this.progress = 0; 
                }, 500);
              }
            }
        },

        async deleteFolder(folderId) {
            if (confirm('Are you sure you want to delete this folder?')) {
              const interval = this.startProgress();
                try {
                    const response = await fetch(`https://fileorganise.onrender.com/api/folders/${folderId}`, {
                        method: 'DELETE',
                        headers: {
                            'Authorization': `Bearer ${localStorage.getItem('token')}`,
                        },
                    });

                    if (response.ok) {
                        this.folders = this.folders.filter(folder => folder._id !== folderId);
                        this.files = this.files.filter(file => file.folder !== folderId);
                        if (this.currentFolder && this.currentFolder._id === folderId) {
                            this.currentFolder = null;
                        }
                        this.progress = 100; 
                    } else {
                        console.error('Error deleting folder:', await response.text());
                    }
                } catch (error) {
                    console.error('Error deleting folder:', error);
                }finally {
                  clearInterval(interval);
                  setTimeout(() => {
                    this.isLoading = false;
                    this.progress = 0;
                  }, 500);
              }
            }
        },
    
        toggleView() {
          this.isGridView = !this.isGridView;
        },
    
        toggleDropdown() {
          this.dropdownVisible = !this.dropdownVisible;
        },
    
        logout() {
          localStorage.removeItem('token');
          this.$router.push('/login');
        },
    
        isImage(fileType) {
          return fileType.startsWith('image/');
        },
    
        isDocument(fileType) {
          return fileType.startsWith('application/') && (fileType.includes('msword') || fileType.includes('vnd.openxmlformats-officedocument.wordprocessingml.document'));
        },
    
        isSpreadsheet(fileType) {
          return fileType.startsWith('application/') && (fileType.includes('vnd.ms-excel') || fileType.includes('vnd.openxmlformats-officedocument.spreadsheetml.sheet'));
        },
    
        isPDF(fileType) {
          return fileType === 'application/pdf';
        },
    
        isCSV(fileType) {
          return fileType === 'text/csv';
        },
    
        isTXT(fileType) {
          return fileType === 'text/plain';
        },
    
        openFolder(folder) {
          this.currentFolder = folder;
        },
    
        goBack() {
          this.currentFolder = null;
        },
    
        // Drag and Drop Methods
        handleDragStart(event, file) {
          this.draggedFile = file;
        },
    
        handleDrop(event, folder) {
          event.preventDefault();
          if (this.draggedFile && folder) {
            this.moveFileToFolder(this.draggedFile._id, folder._id);
            this.draggedFile = null;
          }
        },
        
        handleDragOver(event) {
          event.preventDefault();
        },
    
        async moveFileToFolder(fileId, folderId) {
          if (!fileId || !folderId) {
            console.error('Invalid file ID or folder ID:', fileId, folderId);
            return;
          }

          const interval = this.startProgress(); 

          try {
            const response = await fetch(`https://fileorganise.onrender.com/api/files/${fileId}/move`, {
              method: 'PUT',
              headers: {
                'Authorization': `Bearer ${localStorage.getItem('token')}`,
                'Content-Type': 'application/json',
              },
              body: JSON.stringify({ folderId }),
            });

            if (response.ok) {
              const updatedFile = await response.json();
              this.updateFileInState(updatedFile.file);
              this.progress = 100; 
            } else {
              console.error('Error moving file:', await response.text());
            }
          } catch (error) {
            console.error('Error moving file:', error);
          } finally {
            clearInterval(interval); 
            setTimeout(() => {
              this.isLoading = false;
              this.progress = 0;
            }, 500);
          }
        },

        updateFileInState(updatedFile) {
          // Remove the file from the current folder or top-level files
          if (this.currentFolder) {
            this.currentFolder.files = this.currentFolder.files.filter(file => file._id !== updatedFile._id);
          } else {
            this.files = this.files.filter(file => file._id !== updatedFile._id);
          }
    
          // Add the file to the new folder
          const targetFolder = this.folders.find(folder => folder._id === updatedFile.folder);
          if (targetFolder) {
            targetFolder.files.push(updatedFile);
          } else {
            this.files.push(updatedFile);
          }
        },
        
      },
    
      mounted() {
        this.fetchFolders();
        this.fetchFiles();
      },
    };
    </script>
    
    <style scoped>
    .loader {
      border: 16px solid #f3f3f3; 
      border-top: 16px solid #4F46E5;  
      border-radius: 50%;
      width: 120px;
      height: 120px;
      animation: spin 2s linear infinite;
    }

    @keyframes spin {
      0% { transform: rotate(0deg); }
      100% { transform: rotate(360deg); }
    }
    </style>
