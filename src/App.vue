<template>
  <div id="app">
    <div size="120" class="user">
      <img :src="image_name" class="profile-img" />
      <button @click="$refs.FileInput.click()">Upload</button>

      <input
        ref="FileInput"
        type="file"
        style="display: none;"
        @change="onFileSelect"
      />
    </div>
    <div v-show="dialog" width="500">
      <div>
        <div>
          <VueCropper
            v-show="selectedFile"
            ref="cropper"
            :src="selectedFile"
            alt="Source Image"
            :aspectRatio="5 / 6"
            :initialAspectRatio="1 / 1"
            :rotatable="true"
            :img-style="{ width: '400px', height: '300px' }"
          ></VueCropper>
        </div>
        <div>
          <button class="primary" @click="saveImage(), (dialog = false)">
            Crop
          </button>
          <button @click="rotate">Rotate</button>
          <button color="primary" text @click="dialog = false">Cancel</button>
        </div>
      </div>
    </div>
    <p>
      <!-- https://laracasts.com/discuss/channels/vue/how-to-set-aspect-ratio-to-11-in-vue-cropperjs -->
    </p>
    <p>
      <!-- https://github.com/fengyuanchen/cropperjs#options -->
    </p>
  </div>
</template>
<script>
import VueCropper from 'vue-cropperjs'
import 'cropperjs/dist/cropper.css'
import axios from 'axios'
export default {
  name: 'App',
  props: ['image_name'],
  components: { VueCropper },
  data() {
    return {
      mime_type: '',
      cropedImage: '',
      autoCrop: false,
      selectedFile: '',
      image: '',
      dialog: false,
      files: '',
      imgSrc: ''
    }
  },
  methods: {
    saveImage() {
      // const userId = 12
      this.cropedImage = this.$refs.cropper.getCroppedCanvas().toDataURL()
      this.$refs.cropper.getCroppedCanvas().toBlob(blob => {
        const formData = new FormData()
        const id = 259
        formData.append('id', id)
        formData.append('file', blob, id + '.jpg')
        axios
          .post('/api/', formData)
          .then(response => {
            console.log('saveImage -> response', response)
          })
          .catch(function(error) {
            console.log(error)
          })
      }, this.mime_type)
    },
    onFileSelect(e) {
      const file = e.target.files[0]
      this.mime_type = file.type
      console.log(this.mime_type)
      if (typeof FileReader === 'function') {
        this.dialog = true
        const reader = new FileReader()
        reader.onload = event => {
          this.selectedFile = event.target.result
          this.$refs.cropper.replace(this.selectedFile)
        }
        reader.readAsDataURL(file)
      } else {
        alert('Sorry, FileReader API not supported')
      }
    },
    rotate() {
      // guess what this does :)
      this.$refs.cropper.rotate(90)
    }
  }
}
</script>

<style>
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}
</style>
