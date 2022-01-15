<template>
  <div class="container mx-auto text-center py-5">
    <h1 class="font-medium text-xl">cartesianPNG</h1>
  </div>
  <div class="grid grid-cols-2 grid-rows-1 gap-8 w-auto">
    <div>
      <div class="container mx-auto text-center py-5">
        <button
          class="mx-2 px-6 py-2 bg-sky-500 font-medium text-sm hover:bg-sky-600 text-sky-100 rounded"
          v-if="!exporting && !layers.some(l => l.images.length === 0)"
          @click="addLayer()"
        >+ New Layer</button>
        <button
          class="mx-2 px-6 py-2 bg-lime-500 font-medium text-sm hover:bg-lime-600 text-lime-100 rounded"
          v-if="layers.length > 1 && !exporting"
          @click="generateAllImages()"
        >Generating {{ possibleCombinations }} Images</button>

        <span v-if="exporting === 'generating'">
          <h2
            class="font-medium text-lg"
          >Generating {{ savedImages }} / {{ possibleCombinations }} ...</h2>
        </span>
        <span v-if="exporting === 'zipping'">
          <h2 class="font-medium text-lg">Generating zip file, please wait (~1min)...</h2>
        </span>
        <span v-if="exporting === 'downloading'">
          <h2 class="font-medium text-lg">Download ...</h2>
        </span>
      </div>
      <div class="flex flex-col-reverse">
        <div
          v-for="(layer, index) in layers"
          :key="index"
          class="my-2 px-4 py-4 bg-white rounded-xl shadow-md space-y-2 sm:(py-4 space-y-0 space-x-6)"
        >
          <div class="grid grid-flow-col auto-cols-max">
            <div class="flex flex-col">
              <input
                class="px-2 w-32 py-1 text-lg font-semibold rounded"
                type="text"
                v-model="layer.name"
              />
              <div class="ml-2">
                <input
                  class="form-check-input appearance-none h-4 w-4 border border-gray-300 rounded-sm bg-white checked:bg-sky-600 checked:border-sky-600 focus:outline-none transition duration-200 mt-1 align-top bg-no-repeat bg-center bg-contain float-left mr-2 cursor-pointer"
                  type="checkbox"
                  value
                  :id="`flexCheckDefault${index}`"
                  v-model="layer.optional"
                />
                <label
                  class="form-check-label inline-block text-gray-800"
                  :for="`flexCheckDefault${index}`"
                >optional</label>
              </div>
            </div>
            <div class="dropbox" v-if="layer?.images.length === 0">
              <input
                class="input-file px-2 py-1 text-sm font-semibold rounded hover:(text-white bg-sky-100 border-transparent) focus:(outline-none ring-2 ring-sky-200)"
                type="file"
                @change="addImagesToLayer($event, index)"
                multiple
                accept="image/png"
              />
              <p class="m-5">
                Drag your file(s) here to begin
                <br />or click to browse
              </p>
            </div>
            <div class="m-2">
              <svg
                v-if="index !== layers.length - 1"
                @click="moveLayer(index, 1)"
                aria-hidden="true"
                focusable="false"
                data-prefix="far"
                data-icon="arrow-alt-circle-up"
                class="w-5 h-5"
                role="img"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 512 512"
              >
                <path
                  fill="currentColor"
                  d="M256 504c137 0 248-111 248-248S393 8 256 8 8 119 8 256s111 248 248 248zm0-448c110.5 0 200 89.5 200 200s-89.5 200-200 200S56 366.5 56 256 145.5 56 256 56zm20 328h-40c-6.6 0-12-5.4-12-12V256h-67c-10.7 0-16-12.9-8.5-20.5l99-99c4.7-4.7 12.3-4.7 17 0l99 99c7.6 7.6 2.2 20.5-8.5 20.5h-67v116c0 6.6-5.4 12-12 12z"
                />
              </svg>
              <svg
                v-if="index !== 0"
                @click="moveLayer(index, -1)"
                aria-hidden="true"
                focusable="false"
                data-prefix="far"
                data-icon="arrow-alt-circle-down"
                class="w-5 h-5"
                role="img"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 512 512"
              >
                <path
                  fill="currentColor"
                  d="M256 8C119 8 8 119 8 256s111 248 248 248 248-111 248-248S393 8 256 8zm0 448c-110.5 0-200-89.5-200-200S145.5 56 256 56s200 89.5 200 200-89.5 200-200 200zm-32-316v116h-67c-10.7 0-16 12.9-8.5 20.5l99 99c4.7 4.7 12.3 4.7 17 0l99-99c7.6-7.6 2.2-20.5-8.5-20.5h-67V140c0-6.6-5.4-12-12-12h-40c-6.6 0-12 5.4-12 12z"
                />
              </svg>
              <br />
              <svg
                aria-hidden="true"
                focusable="false"
                data-prefix="far"
                data-icon="times-circle"
                class="svg-inline--fa fa-times-circle fa-w-16"
                role="img"
                xmlns="http://www.w3.org/2000/svg"
                viewBox="0 0 512 512"
                @click="removeLayer(index)"
              >
                <path
                  fill="currentColor"
                  d="M256 8C119 8 8 119 8 256s111 248 248 248 248-111 248-248S393 8 256 8zm0 448c-110.5 0-200-89.5-200-200S145.5 56 256 56s200 89.5 200 200-89.5 200-200 200zm101.8-262.2L295.6 256l62.2 62.2c4.7 4.7 4.7 12.3 0 17l-22.6 22.6c-4.7 4.7-12.3 4.7-17 0L256 295.6l-62.2 62.2c-4.7 4.7-12.3 4.7-17 0l-22.6-22.6c-4.7-4.7-4.7-12.3 0-17l62.2-62.2-62.2-62.2c-4.7-4.7-4.7-12.3 0-17l22.6-22.6c4.7-4.7 12.3-4.7 17 0l62.2 62.2 62.2-62.2c4.7-4.7 12.3-4.7 17 0l22.6 22.6c4.7 4.7 4.7 12.3 0 17z"
                />
              </svg>
            </div>
          </div>
          <section class="py-8 px-4">
            <div class="flex flex-wrap justify-start -mx-4 -mb-8">
              <div
                class="w-1/4 sm:w-1/8 md:w-1/12 mx-3 flex flex-wrap content-center"
                v-for="(image, index) in layer?.images"
              >
                <img class="rounded shadow-md" :src="image.src" />
              </div>
            </div>
          </section>
        </div>
      </div>
    </div>
    <div>
      <h2 class="text-lg font-semibold">Example Image</h2>
      <img v-if="possibleCombinations <= 1"  src="~/assets/img/start_here.png" class="start-img"/>
      <canvas id="exampleCanvas"></canvas>
    </div>
  </div>
</template>

<style>

/* windicss layer base */
*, ::before, ::after {
  box-sizing: border-box;
  border-width: 0;
  border-style: solid;
  border-color: #e5e7eb;
}
* {
  --tw-ring-inset: var(--tw-empty,/*!*/ /*!*/);
  --tw-ring-offset-width: 0px;
  --tw-ring-offset-color: #fff;
  --tw-ring-color: rgba(59, 130, 246, 0.5);
  --tw-ring-offset-shadow: 0 0 #0000;
  --tw-ring-shadow: 0 0 #0000;
  --tw-shadow: 0 0 #0000;
}
:root {
  -moz-tab-size: 4;
  -o-tab-size: 4;
  tab-size: 4;
}
:-moz-focusring {
  outline: 1px dotted ButtonText;
}
:-moz-ui-invalid {
  box-shadow: none;
}
::moz-focus-inner {
  border-style: none;
  padding: 0;
}
::-webkit-inner-spin-button, ::-webkit-outer-spin-button {
  height: auto;
}
::-webkit-search-decoration {
  -webkit-appearance: none;
}
::-webkit-file-upload-button {
  -webkit-appearance: button;
  font: inherit;
}
[type='search'] {
  -webkit-appearance: textfield;
  outline-offset: -2px;
}
abbr[title] {
  -webkit-text-decoration: underline dotted;
  text-decoration: underline dotted;
}
body {
  margin: 0;
  font-family: inherit;
  line-height: inherit;
}
button, input {
  font-family: inherit;
  font-size: 100%;
  line-height: 1.15;
  margin: 0;
  padding: 0;
  line-height: inherit;
  color: inherit;
}
button {
  text-transform: none;
  background-color: transparent;
  background-image: none;
}
button, [type='button'], [type='reset'], [type='submit'] {
  -webkit-appearance: button;
}
button, [role="button"] {
  cursor: pointer;
}
html {
  -webkit-text-size-adjust: 100%;
  font-family: ui-sans-serif,system-ui,-apple-system,BlinkMacSystemFont,"Segoe UI",Roboto,"Helvetica Neue",Arial,"Noto Sans",sans-serif,"Apple Color Emoji","Segoe UI Emoji","Segoe UI Symbol","Noto Color Emoji";
  line-height: 1.5;
}
h1, h2, p {
  margin: 0;
}
h1, h2 {
  font-size: inherit;
  font-weight: inherit;
}
img {
  border-style: solid;
  max-width: 100%;
  height: auto;
}
input::-moz-placeholder {
  opacity: 1;
  color: #9ca3af;
}
input:-ms-input-placeholder {
  opacity: 1;
  color: #9ca3af;
}
input::placeholder {
  opacity: 1;
  color: #9ca3af;
}
input::webkit-input-placeholder {
  opacity: 1;
  color: #9ca3af;
}
input::-moz-placeholder {
  opacity: 1;
  color: #9ca3af;
}
input:-ms-input-placeholder {
  opacity: 1;
  color: #9ca3af;
}
input::-ms-input-placeholder {
  opacity: 1;
  color: #9ca3af;
}
svg, img, canvas {
  display: block;
  vertical-align: middle;
}
/* windicss layer components */
/* windicss layer utilities */
.container {
  width: 100%;
}
@media (min-width: 640px) {
  .container {
    max-width: 640px;
  }
}
@media (min-width: 768px) {
  .container {
    max-width: 768px;
  }
}
@media (min-width: 1024px) {
  .container {
    max-width: 1024px;
  }
}
@media (min-width: 1280px) {
  .container {
    max-width: 1280px;
  }
}
@media (min-width: 1536px) {
  .container {
    max-width: 1536px;
  }
}
.space-y-2 > :not([hidden]) ~ :not([hidden]) {
  --tw-space-y-reverse: 0;
  margin-top: calc(0.5rem * calc(1 - var(--tw-space-y-reverse)));
  margin-bottom: calc(0.5rem * var(--tw-space-y-reverse));
}
.appearance-none {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
}
.bg-sky-500 {
  --tw-bg-opacity: 1;
  background-color: rgba(14, 165, 233, var(--tw-bg-opacity));
}
.hover\:bg-sky-600:hover {
  --tw-bg-opacity: 1;
  background-color: rgba(2, 132, 199, var(--tw-bg-opacity));
}
.bg-lime-500 {
  --tw-bg-opacity: 1;
  background-color: rgba(132, 204, 22, var(--tw-bg-opacity));
}
.hover\:bg-lime-600:hover {
  --tw-bg-opacity: 1;
  background-color: rgba(101, 163, 13, var(--tw-bg-opacity));
}
.bg-white {
  --tw-bg-opacity: 1;
  background-color: rgba(255, 255, 255, var(--tw-bg-opacity));
}
.checked\:bg-sky-600:checked {
  --tw-bg-opacity: 1;
  background-color: rgba(2, 132, 199, var(--tw-bg-opacity));
}
.hover\:bg-sky-100:hover {
  --tw-bg-opacity: 1;
  background-color: rgba(224, 242, 254, var(--tw-bg-opacity));
}
.bg-center {
  background-position: center;
}
.bg-no-repeat {
  background-repeat: no-repeat;
}
.bg-contain {
  background-size: contain;
}
.border-gray-300 {
  --tw-border-opacity: 1;
  border-color: rgba(209, 213, 219, var(--tw-border-opacity));
}
.checked\:border-sky-600:checked {
  --tw-border-opacity: 1;
  border-color: rgba(2, 132, 199, var(--tw-border-opacity));
}
.hover\:border-transparent:hover {
  border-color: transparent;
}
.rounded {
  border-radius: 0.25rem;
}
.rounded-xl {
  border-radius: 0.75rem;
}
.rounded-sm {
  border-radius: 0.125rem;
}
.border {
  border-width: 1px;
}
.cursor-pointer {
  cursor: pointer;
}
.inline-block {
  display: inline-block;
}
.flex {
  display: flex;
}
.grid {
  display: -ms-grid;
  display: grid;
}
.flex-col {
  flex-direction: column;
}
.flex-col-reverse {
  flex-direction: column-reverse;
}
.flex-wrap {
  flex-wrap: wrap;
}
.content-center {
  align-content: center;
}
.justify-start {
  justify-content: flex-start;
}
.float-left {
  float: left;
}
.font-medium {
  font-weight: 500;
}
.font-semibold {
  font-weight: 600;
}
.h-4 {
  height: 1rem;
}
.h-5 {
  height: 1.25rem;
}
.text-xl {
  font-size: 1.25rem;
  line-height: 1.75rem;
}
.text-sm {
  font-size: 0.875rem;
  line-height: 1.25rem;
}
.text-lg {
  font-size: 1.125rem;
  line-height: 1.75rem;
}
.m-5 {
  margin: 1.25rem;
}
.m-2 {
  margin: 0.5rem;
}
.mx-auto {
  margin-left: auto;
  margin-right: auto;
}
.mx-2 {
  margin-left: 0.5rem;
  margin-right: 0.5rem;
}
.my-2 {
  margin-top: 0.5rem;
  margin-bottom: 0.5rem;
}
.-mx-4 {
  margin-left: -1rem;
  margin-right: -1rem;
}
.mx-3 {
  margin-left: 0.75rem;
  margin-right: 0.75rem;
}
.ml-2 {
  margin-left: 0.5rem;
}
.mt-1 {
  margin-top: 0.25rem;
}
.mr-2 {
  margin-right: 0.5rem;
}
.-mb-8 {
  margin-bottom: -2rem;
}
.focus\:outline-none:focus {
  outline: 2px solid transparent;
  outline-offset: 2px;
}
.py-5 {
  padding-top: 1.25rem;
  padding-bottom: 1.25rem;
}
.px-6 {
  padding-left: 1.5rem;
  padding-right: 1.5rem;
}
.py-2 {
  padding-top: 0.5rem;
  padding-bottom: 0.5rem;
}
.px-4 {
  padding-left: 1rem;
  padding-right: 1rem;
}
.py-4 {
  padding-top: 1rem;
  padding-bottom: 1rem;
}
.px-2 {
  padding-left: 0.5rem;
  padding-right: 0.5rem;
}
.py-1 {
  padding-top: 0.25rem;
  padding-bottom: 0.25rem;
}
.py-8 {
  padding-top: 2rem;
  padding-bottom: 2rem;
}
.shadow-md {
  --tw-shadow: 0 4px 6px -1px rgb(0 0 0/0.1),0 2px 4px -2px rgb(0 0 0/0.1);
  --tw-shadow-colored: 0 4px 6px -1px var(--tw-shadow-color),0 2px 4px -2px var(--tw-shadow-color);
  box-shadow: var(--tw-ring-offset-shadow,0 0 #0000),var(--tw-ring-shadow,0 0 #0000),var(--tw-shadow);
}
.focus\:ring-2:focus {
  --tw-ring-offset-shadow: var(--tw-ring-inset) 0 0 0 var(--tw-ring-offset-width) var(--tw-ring-offset-color);
  --tw-ring-shadow: var(--tw-ring-inset) 0 0 0 calc(2px + var(--tw-ring-offset-width)) var(--tw-ring-color);
  box-shadow: var(--tw-ring-offset-shadow), var(--tw-ring-shadow), var(--tw-shadow, 0 0 #0000);
}
.focus\:ring-sky-200:focus {
  --tw-ring-opacity: 1;
  --tw-ring-color: rgba(186, 230, 253, var(--tw-ring-opacity));
}
.text-center {
  text-align: center;
}
.text-sky-100 {
  --tw-text-opacity: 1;
  color: rgba(224, 242, 254, var(--tw-text-opacity));
}
.text-lime-100 {
  --tw-text-opacity: 1;
  color: rgba(236, 252, 203, var(--tw-text-opacity));
}
.text-gray-800 {
  --tw-text-opacity: 1;
  color: rgba(31, 41, 55, var(--tw-text-opacity));
}
.hover\:text-white:hover {
  --tw-text-opacity: 1;
  color: rgba(255, 255, 255, var(--tw-text-opacity));
}
.align-top {
  vertical-align: top;
}
.invisible {
  visibility: hidden;
}
.w-auto {
  width: auto;
}
.w-32 {
  width: 8rem;
}
.w-4 {
  width: 1rem;
}
.w-5 {
  width: 1.25rem;
}
.w-1\/4 {
  width: 25%;
}
.gap-8 {
  grid-gap: 2rem;
  gap: 2rem;
}
.grid-flow-col {
  grid-auto-flow: column;
}
.grid-cols-2 {
  grid-template-columns: repeat(2, minmax(0, 1fr));
}
.auto-cols-max {
  grid-auto-columns: -webkit-max-content;
  grid-auto-columns: max-content;
}
.grid-rows-1 {
  grid-template-rows: repeat(1, minmax(0, 1fr));
}
.transition {
  transition-property: background-color, border-color, color, fill, stroke, opacity, box-shadow, transform, filter, -webkit-backdrop-filter;
  transition-property: background-color, border-color, color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter;
  transition-property: background-color, border-color, color, fill, stroke, opacity, box-shadow, transform, filter, backdrop-filter, -webkit-backdrop-filter;
  transition-timing-function: cubic-bezier(0.4, 0, 0.2, 1);
  transition-duration: 150ms;
}
.duration-200 {
  transition-duration: 200ms;
}
@media (min-width: 640px) {
  .sm\:space-y-0 > :not([hidden]) ~ :not([hidden]) {
    --tw-space-y-reverse: 0;
    margin-top: calc(0px * calc(1 - var(--tw-space-y-reverse)));
    margin-bottom: calc(0px * var(--tw-space-y-reverse));
  }
  .sm\:space-x-6 > :not([hidden]) ~ :not([hidden]) {
    --tw-space-x-reverse: 0;
    margin-right: calc(1.5rem * var(--tw-space-x-reverse));
    margin-left: calc(1.5rem * calc(1 - var(--tw-space-x-reverse)));
  }
  .sm\:py-4 {
    padding-top: 1rem;
    padding-bottom: 1rem;
  }
  .sm\:w-1\/8 {
    width: 12.5%;
  }
}
@media (min-width: 768px) {
  .md\:w-1\/12 {
    width: 8.333333%;
  }
}

.dropbox {
  outline: 2px dashed grey; /* the dash box */
  outline-offset: -10px;
  background: #fff;
  color: dimgray;
  padding: 10px 10px;
  min-height: 150px; /* minimum height */
  position: relative;
  cursor: pointer;
}

.start-img {
  max-width: 100%;
  margin: 30px;
}

.input-file {
  opacity: 0; /* invisible but it's there! */
  width: 200px;
  height: 150px;
  position: absolute;
  cursor: pointer;
}

.dropbox:hover {
  background: #f3f3f3; /* when mouse over to the drop zone, change color */
}

.dropbox p {
  text-align: center;
  padding: 25px 10px;
}
</style>

<script>
import JSZip from "jszip";
import { saveAs } from 'file-saver';

class Layer {
  constructor(name) {
    this.name = name;
    this.optional = false;
    this.images = [];
  }
}


export default {
  setup(props) {
    let exampleCanvas;
    let exampleCtx;
    let exporting = ref("");
    let savedImages = ref(0);
    const layers = reactive([new Layer("Layer #0")]);

    onMounted(() => {
      exampleCanvas = document.getElementById("exampleCanvas");
      exampleCanvas.width = 1;
      exampleCanvas.height = 1;
      exampleCtx = exampleCanvas.getContext("2d");
    });
    const addImageToCanvas = (image) => {
      if (!image) {
        return;
      }
      if (exampleCanvas.width !== image.width || exampleCanvas.height !== image.height) {
        exampleCanvas.width = image.width;
        exampleCanvas.height = image.height;
      }
      exampleCtx.drawImage(image, 0, 0);
    }
    const addImagesToLayer = async ($event, index) => {
      layers[index].images = [];
      let allImagesLoadedPromise = [];
      for (const file of $event?.target?.files) {
        const image = new Image();
        image.src = URL.createObjectURL(file);
        layers[index].images.push(image);
        allImagesLoadedPromise.push(new Promise(function (resolve, reject) {
          image.onload = resolve;
        }));
      }
      await Promise.all(allImagesLoadedPromise);
      randomCompleteImage();
      addLayer();
    }

    function* cartesian(head, ...tail) {
      const remainder = tail.length > 0 ? cartesian(...tail) : [[]];
      for (let r of remainder) {
        for (let h of head) {
          yield [h, ...r];
        }
      }
    }

    const allCombinations = computed(() => {
      let onlyImages = [];
      for (const layer of layers) {
        if (layer.images.length > 0) {
          let imageLayer = [...layer.images.values()];
          if (layer.optional) {
            imageLayer.push(null);
          }
          onlyImages.push(imageLayer);
        }
      }
      // remove FALSE optionals
      return [...cartesian(...onlyImages)];
    })

    watch(layers, () => {
      randomCompleteImage();
    })

    const generateAllImages = async () => {
      const zip = new JSZip();
      exporting.value = "generating";
      savedImages.value = 0;
      for (const imageSet of allCombinations.value) {
        clearCtx();
        for (const image of imageSet) {
          if (image !== null) {
            addImageToCanvas(image);
          }
        }
        await new Promise((resolve, reject) => {
          exampleCanvas.toBlob((blob) => {
            const filename = [...Array(30)].map(() => Math.random().toString(36)[2]).join('');
            zip.file(`${filename}.png`, blob);
            savedImages.value++;
            resolve(savedImages.value);
          }, 'image/png');
        })
      }
      exporting.value = "zipping";
      const d = new Date();
      const dateString = d.toLocaleString().replace(/\.:, /ig, '_');
      zip.generateAsync({ type: "blob" }).then(async (blob) => {
        exporting.value = "downloading";
        await saveAs(blob, `cartesianPNG_${dateString}.zip`);
        exporting.value = "";
      }, (err) => {
        console.log(err);
      });

      randomCompleteImage();
    }

    const randomCompleteImage = () => {
      clearCtx();
      for (const layer of layers) {
        const image = layer.images[Math.floor(Math.random() * layer.images.length)];
        addImageToCanvas(image);
      }
    }

    const clearCtx = () => {
      exampleCtx.clearRect(0, 0, exampleCanvas.width, exampleCanvas.height);
    }

    const possibleCombinations = computed(() => {
      return layers.reduce((all, curr) => {
        return all *= curr.images.length > 0 ? curr.images.length + (~~curr.optional) : 1;
      }, ~~!!layers.length);
    })

    const addLayer = () => {
      layers.push(new Layer(`Layer #${layers.length}`))
    }
    const removeLayer = (index) => {
      layers.splice(index, 1);
      renameLayers();
    }
    const moveLayer = (index, direction) => {
      [layers[index], layers[index + direction]] = [layers[index + direction], layers[index]];
      renameLayers();
    }
    const renameLayers = () => {
      for (let [index, layer] of layers.entries()) {
        console.log(index);
        layer.name = layer.name.replace(/^(Layer #)(?:\d+?)/i, `$1${index}`);
      }
    }
    return { layers, addLayer, addImagesToLayer, removeLayer, moveLayer, possibleCombinations, allCombinations, generateAllImages, exporting, savedImages }
  }
}

</script>