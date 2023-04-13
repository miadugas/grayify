<template>
  <div>
    <section class="bg-gray-50 dark:bg-gray-900">
      <div
        class="py-8 px-4 mx-auto max-w-screen-xl lg:py-16 grid lg:grid-cols-2 gap-8 lg:gap-16"
      >
        <div class="flex flex-col justify-center">
          <h1
            class="mb-4 text-4xl font-extrabold tracking-tight leading-none text-gray-900 md:text-5xl lg:text-6xl dark:text-white"
          >
            Grayify
          </h1>
          <p
            class="mb-6 text-lg font-normal text-gray-500 lg:text-xl dark:text-gray-400"
          >
            convert and adjust color pictures to grayscale
          </p>
          <!-- old -->
          <canvas ref="canvas" class="my-4 border border-gray-400">
            <p v-if="!imageLoaded" class="text-center">
              Upload an image to preview it here
            </p>
          </canvas>
          <!-- old end -->
        </div>

        <div>
          <div
            class="w-full lg:max-w-xl p-6 space-y-8 sm:p-8 bg-white rounded-lg shadow-xl dark:bg-gray-800"
          >
            <h2 class="text-2xl font-bold text-gray-900 dark:text-white">
              <label for="fileInput">Select image file:</label>
            </h2>
            <div class="my-4">
              <input
                class="px-3 border rounded w-full"
                type="file"
                id="fileInput"
                @change="onFileSelected"
              />
            </div>
            <!-- Width -->
            <div class="my-4">
              <label class="block font-medium mb-1" for="canvasWidth"
                >Canvas width:</label
              >
              <input
                class="py-2 px-3 border rounded w-full"
                type="number"
                id="canvasWidth"
                v-model.number="canvasWidth"
              />
            </div>
            <!-- old end -->
            <!-- Height -->
            <div class="my-4">
              <label class="block font-medium mb-1" for="canvasHeight"
                >Canvas height:</label
              >
              <input
                class="py-2 px-3 border rounded w-full"
                type="number"
                id="canvasHeight"
                v-model.number="canvasHeight"
              />
            </div>
            <!-- old end -->
            <!-- file type -->
            <div class="my-4">
              <label class="block font-medium mb-1" for="fileType"
                >File type:</label
              >
              <select
                class="py-2 px-3 border rounded w-full"
                id="fileType"
                v-model="fileType"
              >
                <option value="jpeg">JPEG</option>
                <option value="png">PNG</option>
              </select>
            </div>
            <!-- old end -->
            <!-- Image size -->
            <div class="my-4">
              <label class="block font-medium mb-1" for="imageSize"
                >Image size:</label
              >
              <input
                class="py-2 px-3 border rounded w-full"
                type="text"
                id="imageSize"
                :value="imageSize"
                disabled
              />
            </div>

            <!-- old -->
            <div class="flex my-4 space-x-4">
              <button
                v-if="imageLoaded"
                @click="convertToGrayscale"
                class="py-2 px-4 bg-green-500 text-white rounded hover:bg-green-600"
              >
                Convert to grayscale
              </button>

              <button
                v-if="imageLoaded"
                @click="resetCanvas"
                class="py-2 px-4 bg-red-500 text-white rounded hover:bg-red-600"
              >
                Reset
              </button>
            </div>

            <!-- old end -->

            <!-- old -->
            <div v-if="progress > 0" class="my-4">
              <progress
                :max="100"
                :value="progress"
                class="w-full h-4 rounded-full overflow-hidden"
              >
                <div
                  class="bg-gray-300 h-full rounded-full"
                  :style="{ width: `${progress}%` }"
                ></div>
              </progress>
              <p class="mt-2">{{ progress }}% complete</p>
            </div>
            <!-- old end -->
            <!-- old -->
            <div v-if="outputDataURL" class="my-4">
              <button
                @click="saveImageLocally"
                class="my-4 py-2 px-4 bg-gray-800 text-white rounded hover:bg-gray-700"
              >
                Save image
              </button>
              <a
                :href="outputDataURL"
                :download="outputFileName"
                class="ml-4 py-2 px-4 bg-gray-800 text-white rounded hover:bg-gray-700"
                >Download image</a
              >
            </div>
          </div>
        </div>
      </div>
    </section>
  </div>
</template>

<script>
export default {
  name: 'Grayify',
  data() {
    return {
      imageLoaded: false,
      progress: 0,
      canvasWidth: 0, // default value
      canvasHeight: 0, // default value
      fileType: 'jpeg',
      outputDataURL: null,
      outputFileName: null,
      imageSize: '',
    };
  },
  methods: {
    onFileSelected(event) {
      const file = event.target.files[0];
      this.imageSize = (file.size / 1024).toFixed(2) + ' KB';
      if (file.size / (1024 * 1024) > 1) {
        this.imageSize = (file.size / (1024 * 1024)).toFixed(2) + ' MB';
      }
      const reader = new FileReader();
      reader.readAsDataURL(file);
      reader.onload = () => {
        const img = new Image();

        img.onload = () => {
          const canvas = this.$refs.canvas;
          const ctx = canvas.getContext('2d');
          const scale = Math.min(
            canvas.width / img.width,
            canvas.height / img.height
          );
          const width = img.width * scale;
          const height = img.height * scale;
          canvas.width = width;
          canvas.height = height;
          ctx.drawImage(img, 0, 0, width, height);
          this.imageLoaded = true;

          // Set the current width and height values
          this.canvasWidth = img.width;
          this.canvasHeight = img.height;
        };
        img.src = reader.result;
      };
    },
    convertToGrayscale() {
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      const imageData = ctx.getImageData(0, 0, canvas.width, canvas.height);
      const data = imageData.data;
      const numPixels = data.length / 4;
      let grayscaleData = new Uint8ClampedArray(numPixels * 4);
      let j = 0;
      for (let i = 0; i < data.length; i += 4) {
        const red = data[i];
        const green = data[i + 1];
        const blue = data[i + 2];
        const gray = Math.floor(0.2126 * red + 0.7152 * green + 0.0722 * blue);
        grayscaleData[j] = gray;
        grayscaleData[j + 1] = gray;
        grayscaleData[j + 2] = gray;
        grayscaleData[j + 3] = 255;
        j += 4;
      }
      const grayscaleImageData = new ImageData(
        grayscaleData,
        canvas.width,
        canvas.height
      );
      ctx.putImageData(grayscaleImageData, 0, 0);
      const outputDataURL = canvas.toDataURL(`image/${this.fileType}`);
      this.outputDataURL = outputDataURL;
      this.outputFileName = `grayscale.${this.fileType}`;
    },
    saveImageLocally() {
      const link = document.createElement('a');
      link.download = this.outputFileName;
      link.href = this.outputDataURL;
      link.click();
    },
    resetCanvas() {
      location.reload();
      const canvas = this.$refs.canvas;
      const ctx = canvas.getContext('2d');
      ctx.clearRect(0, 0, canvas.width, canvas.height);

      this.imageLoaded = false;
      this.progress = 0;
      this.canvasWidth = 0;
      this.canvasHeight = 0;
      this.fileType = 'jpeg';
      this.outputDataURL = null;
      this.outputFileName = null;
      this.imageSize = '';

      const fileInput = document.getElementById('fileInput');
      fileInput.value = null;
    },
  },
};
</script>

<style scoped></style>
