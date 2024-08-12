<template>
  <div id="app">
    <h1>Dynamic Image Gallery</h1>
    
    <div class="upload-container">
      <input type="file" @change="onFileChange" multiple class="upload-input" />
      <button @click="uploadImages" class="upload-button">Upload</button>
    </div>
    
    <div v-if="uploading" class="upload-progress-container">
      <label for="uploadProgress">Upload Progress:</label>
      <input 
        type="range" 
        id="uploadProgress" 
        min="0" 
        max="100" 
        :value="uploadProgress" 
        disabled 
      />
      <span>{{ uploadProgress }}%</span>
    </div>
    
    <div class="gallery">
      <div v-for="(image, index) in images" :key="index" class="image-item">
        <div class="image-wrapper">
          <img 
            :src="image.url" 
            :alt="'Image ' + index" 
            class="image-preview" 
            @click="showImage(index)" />
          <button class="delete-btn" @click="deleteImage(index)">✖</button>
        </div>
        <div class="image-name">{{ image.name }}</div>
      </div>
    </div>

    <div v-if="showModal" class="modal" @click.self="closeModal">
      <div class="modal-content">
        <span class="close" @click="closeModal">&times;</span>
        <button class="nav-button first" @click="showFirstImage">&lt;&lt;</button>
        <button class="nav-button prev" @click="showPreviousImage">&lt;</button>
        <img :src="selectedImage.url" class="modal-image" />
        <button class="nav-button next" @click="showNextImage">&gt;</button>
        <button class="nav-button last" @click="showLastImage">&gt;&gt;</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      images: [
      { url: require('/home/san/Pictures/img/img_2.png'), name: 'img2' },
      // { url: require('C:/Users/Student/Pictures/thiru.png'), name: 'thiru' },
      //{ url: require('C:/Users/Student/Pictures/thiru.png'), name: 'thiru' }
      { url: require('/home/san/Pictures/img/img_3.png'), name: 'img3' }
      ],
      selectedFiles: [],
      uploadProgress: 0,
      uploading: false,
      showModal: false,
      selectedImageIndex: null
    };
  },
  computed: {
    selectedImage() {
      return this.images[this.selectedImageIndex] || {};
    }
  },
  methods: {
    onFileChange(event) {
      this.selectedFiles = Array.from(event.target.files);
    },
    async uploadImages() {
      if (this.selectedFiles.length === 0) {
        alert('No files selected!');
        return;
      }
      this.uploading = true;
      this.uploadProgress = 0;
      for (let i = 0; i < this.selectedFiles.length; i++) {
        const file = this.selectedFiles[i];
        await new Promise((resolve) => {
          let progress = 0;
          const interval = setInterval(() => {
            progress += 10;
            this.uploadProgress = Math.min(progress, 100);
            if (progress >= 100) {
              clearInterval(interval);
              resolve();
            }
          }, 200);
        });
        const imageUrl = URL.createObjectURL(file);
        this.images.push({ url: imageUrl, name: file.name });
      }
      this.uploading = false;
      this.uploadProgress = 0;
      this.selectedFiles = [];
    },
    deleteImage(index) {
      URL.revokeObjectURL(this.images[index].url);
      this.images.splice(index, 1);
      if (this.selectedImageIndex !== null && index <= this.selectedImageIndex) {
        this.selectedImageIndex = Math.max(0, this.selectedImageIndex - 1);
      }
    },
    showImage(index) {
      this.selectedImageIndex = index;
      this.showModal = true;
    },
    closeModal() {
      this.showModal = false;
      this.selectedImageIndex = null;
    },
    showPreviousImage() {
      if (this.selectedImageIndex > 0) {
        this.selectedImageIndex--;
      }
    },
    showNextImage() {
      if (this.selectedImageIndex < this.images.length - 1) {
        this.selectedImageIndex++;
      }
    },
    showFirstImage() {
      if (this.images.length > 0) {
        this.selectedImageIndex = 0;
      }
    },
    showLastImage() {
      if (this.images.length > 0) {
        this.selectedImageIndex = this.images.length - 1;
      }
    }
  }
};
</script>

<style scoped>
#app {
  font-family: 'Arial', sans-serif;
  max-width: 900px;
  margin: 20px auto;
  text-align: center;
  background-color: #f5f5f5;
  padding: 20px;
  border-radius: 10px;
  box-shadow: 0 0 10px rgba(0, 0, 0, 0.1);
}

h1 {
  color: #333;
  margin-bottom: 20px;
}

.upload-container {
  margin-bottom: 20px;
}

.upload-input {
  margin-right: 10px;
}

.upload-button {
  padding: 8px 15px;
  background-color: #28a745;
  color: white;
  border: none;
  border-radius: 5px;
  cursor: pointer;
}

.upload-button:hover {
  background-color: #218838;
}

.upload-progress-container {
  margin: 20px 0;
}

.gallery {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  gap: 15px;
}

.image-item {
  position: relative;
  text-align: center;
  cursor: pointer;
}

.image-wrapper {
  position: relative;
  overflow: hidden;
  border-radius: 10px;
  box-shadow: 0 2px 5px rgba(0, 0, 0, 0.2);
}

.image-preview {
  width: 100%;
  height: auto;
  transition: transform 0.3s ease;
}

.image-preview:hover {
  transform: scale(1.05);
}

.delete-btn {
  position: absolute;
  top: 8px;
  right: 8px;
  background-color: rgba(255, 0, 0, 0.7);
  color: white;
  border: none;
  border-radius: 50%;
  padding: 5px;
  cursor: pointer;
}

.delete-btn:hover {
  background-color: red;
}

.image-name {
  margin-top: 10px;
  font-size: 14px;
  color: #666;
}

.modal {
  position: fixed;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  background-color: rgba(0, 0, 0, 0.8);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 1000;
}

.modal-content {
  position: relative;
  max-width: 90%;
  max-height: 90%;
}

.modal-image {
  max-width: 100%;
  max-height: 100%;
  border-radius: 10px;
}

.close {
  position: absolute;
  top: 10px;
  right: 10px;
  color: white;
  font-size: 24px;
  cursor: pointer;
}

.nav-button {
  background-color: rgba(0, 0, 0, 0.5);
  color: white;
  border: none;
  padding: 10px;
  cursor: pointer;
  position: absolute;
  top: 50%;
  transform: translateY(-50%);
  z-index: 1001;
}

.first {
  left: 10%;
}

.prev {
  left: 0;
}

.next {
  right: 0;
}

.last {
  right: 10%;
}

.nav-button:hover {
  background-color: rgba(0, 0, 0, 0.8);
}
</style>
