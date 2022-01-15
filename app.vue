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