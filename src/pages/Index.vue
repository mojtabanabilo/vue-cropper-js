<template>
  <div class="fit column justify-center items-center" dir="rtl">
    <transition name="fade">
      <div v-if="!showModal">
        <input
          type="file"
          ref="upload_image"
          id="getImage"
          @change="handleFileChange"
          class="upload-input"
        />
        <label for="getImage" class="upload-button">+</label>
      </div>
    </transition>

    <transition name="fade">
      <template v-if="showModal">
        <div class="q-pa-lg bg-grey-5 rounded-borders">
          <div class="row no-wrap justify-between items-center q-gutter-md">
            <div class="button-column column q-py-md q-gutter-sm">
              <q-btn label="حالت اولیه" color="primary" @click="restoreHandler" class="blue-button text-subtitle2" :disable="cropimage ? true : false"/>
              <q-btn label="قرینه" color="primary" @click="flipHandler" class="blue-button" :disable="cropimage ? true : false"/>
              <q-btn icon="rotate_right" color="primary" @click="rotateHandler" class="blue-button" :disable="cropimage ? true : false"/>
              <q-btn icon="flip" color="primary" @click="flipHandler" class="blue-button" :disable="cropimage ? true : false"/>
            </div>
            <vue-cropper
              :key="key"
              ref="cropper"
              :src="imageUrl"
              :aspect-ratio="1"
              :restore="restore"
              :movable="true"
              :drag-mode="'move'"
              @ready="onReady"
              class="cropper"
            />
          </div>
          <div class="q-py-md q-gutter-sm">
            <q-btn label="بازگشت" color="primary" @click="backToCrop" class="blue-button" v-if="cropimage"/>
            <q-btn label="برش تصویر" color="primary" @click="cropImageHandler" class="blue-button" v-if="!cropimage"/>
            <q-btn label="لغو" color="negative" @click="cencelHandler" class="blue-button" />
          </div>
          <div v-if="cropimage" class="column">
            <div class="row justify-between items-center">
                <div class="column justify-center items-center">
                  <div class="input-container">
                    <div class="row justify-between items-center full-width">
                      <span>{{ filterCroppedImage.hue }}</span>
                      <label for="hue-shift">تغییر رنگ</label>
                    </div>
                    <div class="row justify-between items-center">
                      <input type="range" name="hue" min="0" max="360" value="0" @input="(e) => filterHandler(e)" v-model="filterCroppedImage.hue">
                      <q-btn round color="red" size="xs" :disable="filterCroppedImage.hue === '0' ? true : false" name="hue" @click="(e) => resetFilterHandler(e)">
                        <q-icon name="close" color="white" size="sm" class="absolute-top-right" />
                      </q-btn>
                    </div>
                  </div>
                  <div class="input-container">
                    <div class="row justify-between items-center full-width">
                      <span>{{ filterCroppedImage.saturation }}</span>
                      <label for="saturation">اشباع رنگ</label>
                    </div>
                    <div class="row justify-between items-center">
                      <input type="range" name="saturation" min="0" max="200" value="100" @input="(e) => filterHandler(e)" v-model="filterCroppedImage.saturation">
                      <q-btn round color="red" size="xs" :disable="filterCroppedImage.saturation === '100' ? true : false" name="saturation" @click="(e) => resetFilterHandler(e)">
                        <q-icon name="close" color="white" size="sm" class="absolute-top-right" />
                      </q-btn>
                    </div>
                  </div>
                  <div class="input-container">
                    <div class="row justify-between items-center full-width">
                      <span>{{ filterCroppedImage.contrast }}</span>
                      <label for="contrast">کنتراست</label>
                    </div>
                    <div class="row justify-between items-center">
                      <input type="range" name="contrast" min="0" max="200" value="100" @input="(e) => filterHandler(e)" v-model="filterCroppedImage.contrast">
                      <q-btn round color="red" size="xs" :disable="filterCroppedImage.contrast === '100' ? true : false" name="contrast" @click="(e) => resetFilterHandler(e)">
                        <q-icon name="close" color="white" size="sm" class="absolute-top-right" />
                      </q-btn>
                    </div>
                  </div>
                  <div class="input-container">
                    <div class="row justify-between items-center full-width">
                      <span>{{ filterCroppedImage.invert_colors }}</span>
                      <label for="invert_colors">معکوس کردن</label>
                    </div>
                    <div class="row justify-between items-center">
                      <input type="range" name="invert_colors" min="0" max="100" value="0" @input="(e) => filterHandler(e)" v-model="filterCroppedImage.invert_colors">
                      <q-btn round color="red" size="xs" :disable="filterCroppedImage.invert_colors === '0' ? true : false" name="invert_colors" @click="(e) => resetFilterHandler(e)">
                        <q-icon name="close" color="white" size="sm" class="absolute-top-right" />
                      </q-btn>
                    </div>
                  </div>
                </div>
              <img :src="cropimage" class="preview-image q-my-md" :style="styleImage"/>
            </div>
            <q-btn
              label="تایید نهایی"
              color="positive"
              class="submit-button q-mt-xs"
            />
          </div>
        </div>
      </template>
    </transition>
  </div>
</template>

<script>
import VueCropper from "vue-cropperjs";
import "cropperjs/dist/cropper.css";

export default {
  components: {
    VueCropper,
  },

  data() {
    return {
      imageUrl: null, // مسیر تصویر ورودی
      cropimage: null, // ذخیره تصویر کراپ‌شده
      uploadedImage: null, //تصویر آپلود شده کاربر
      cancel: false, // کنسل کردن ویرایش عکس
      showModal: false, // نمایش مدال ویرایش عکس
      filterCroppedImage: {
        hue: "0",
        saturation: "100",
        contrast: "100",
        invert_colors: "0"
      }, // فیلتر های عکس کراپ شده

      move_grabber_icon: "/icons8-move-grabber-48.png",
      rotate_icon: "/icons8-rotate-48.png",

      key: 0,
      vertical: false,
      restore: false,
      flip: false,
    };
  },

  computed: {
    styleImage() {
      return {
        filter: `hue-rotate(${this.filterCroppedImage.hue}deg) saturate(${this.filterCroppedImage.saturation}%) contrast(${this.filterCroppedImage.contrast}%) invert(${this.filterCroppedImage.invert_colors}%)`,
      }
    }
  },

  methods: {
    onReady() {
      console.log("Cropper آماده است!");
    },

    cropImageHandler() {
      this.cropimage = this.$refs.cropper.getCroppedCanvas().toDataURL();
    },

    rotateHandler() {
      this.$refs.cropper.rotate(-90);
    },

    restoreHandler() {
      this.restore = true;
      this.key += 1;
    },

    scaleHandler() {
      this.vertical = !this.vertical;
    },

    flipHandler() {
      this.flip = !this.flip;
    },

    handleFileChange(e) {
      this.showModal = true
      this.uploadedImage = e.target.files[0];
      if (this.uploadedImage) {
        const reader = new FileReader();
        reader.onload = (e) => {
          this.imageUrl = e.target.result;
          this.key += 1;
        };
        reader.readAsDataURL(this.uploadedImage);
      }
    },

    cencelHandler() {
      this.imageUrl = null;
      this.cropimage = null;
      this.showModal = false
    },

    backToCrop() {
      this.cropimage = null
      this.filterCroppedImage = {
        hue: "0",
        saturation: "100",
        contrast: "100",
        invert_colors: "0"
      }
    },

    filterHandler(e) {
      this.filterCroppedImage = { ...this.filterCroppedImage, [e.target.name]: e.target.value }
    },

    resetFilterHandler(e) {
      if (e.currentTarget.getAttribute('name') === "hue" || e.currentTarget.getAttribute('name') === "invert_colors") {
        this.filterCroppedImage = { ...this.filterCroppedImage, [e.currentTarget.getAttribute('name')]: "0"}
      } else {
        this.filterCroppedImage = { ...this.filterCroppedImage, [e.currentTarget.getAttribute('name')]: "100"}
      }
      console.log(this.filterCroppedImage, e.currentTarget.getAttribute('name'))
    }
  },

  watch: {
    vertical(newVal) {
      newVal ? this.$refs.cropper.scaleY(-1) : this.$refs.cropper.scaleY(1);
    },

    flip(newVal) {
      newVal ? this.$refs.cropper.scaleX(-1) : this.$refs.cropper.scaleX(1);
    },
  },
};
</script>

<style scoped>
.fade-enter-active, .fade-leave-active {
  transition: opacity .6s ease-in-out;
}
.fade-enter, .fade-leave-to, .fade-leave-active {
  opacity: 0;
}

.button-column {
  min-width: 120px;
}

.cropper {
  max-width: 400px;
  width: 100%;
  height: auto;
}

.upload-input {
  display: none;
}

.upload-button {
  background-color: #007acc;
  color: white;
  font-size: 20px;
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 10px;
  cursor: pointer;
  box-shadow: 0px 2px 8px rgba(0, 0, 0, 0.2);
}

.upload-button:hover {
  background-color: #005f9e;
}

.blue-button {
  font-size: 14px;
  border-radius: 5px;
  cursor: pointer;
}

.preview-image {
  width: 200px;
  height: 200px;
  border: 2px solid #007acc;
  border-radius: 8px;
}

.submit-button {
  color: white;
  background-color: #007a2f;
  font-size: 16px;
}

.submit-button:hover {
  background-color: #01aa42;
}

.input-container{
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-end;
  max-width: 100%;
  direction: ltr;
}

.input-container label{
  font-weight: bold;
  font-size: 1rem;
  color: #2b2b2b;
  margin: 5px 0 0 0;
}
</style>