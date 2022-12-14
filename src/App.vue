<template>
  <div class="template-box">
    <Head title="Camera" />
    <div class="content-box">
      <div id="title-container">마스크를 쓰셨나요?</div>

      <div id="webcam-container"></div>
      <div id="label-container"></div>
      <div id="result-container"></div>
    </div>
  </div>
</template>

<script>
import { Head } from "@inertiajs/inertia-vue";
import "@tensorflow/tfjs";
import * as tmImage from "@teachablemachine/image";

export default {
  components: {
    Head,
  },
  props: {
    products: {
      type: Object,
      default: () => {
        return {};
      },
    },
  },
  meta: [
    { name: "description", content: "description hoge" },
    {
      name: "viewport",
      content: "width=device-width,initial-scale=1.0,user-scalable=no",
    },
    { charset: "utf-8" },
    { property: "og:type", content: "website" },
    // ...
  ],
  data() {
    return {
      model: null,
      webcam: null,
      labelContainer: null,
      maxPredictions: null,
      isIos: false,
    };
  },
  beforeCreate() {},
  created() {},
  beforeMount() {},
  mounted() {
    this.init();
  },
  beforeUpdate() {},
  update() {},
  beforeDestroy() {},
  destroyed() {},
  methods: {
    // Load the image model and setup the webcam
    async init() {
      const URL = "https://teachablemachine.withgoogle.com/models/27xOuCu9f/";

      const modelURL = URL + "model.json";
      const metadataURL = URL + "metadata.json";

      // fix when running demo in ios, video will be frozen;
      if (
        window.navigator.userAgent.indexOf("iPhone") > -1 ||
        window.navigator.userAgent.indexOf("iPad") > -1
      ) {
        this.isIos = true;
      }

      // load the model and metadata
      // Refer to tmImage.loadFromFiles() in the API to support files from a file picker
      // or files from your local hard drive
      this.model = await tmImage.load(modelURL, metadataURL);
      this.maxPredictions = this.model.getTotalClasses();

      // convenience function to setup a webcam
      const width = 250;
      const height = 250;
      const flip = true; // whether to flip the webcam
      this.webcam = new tmImage.Webcam(width, height, flip); // width, height, flip
      await this.webcam.setup({ facingMode: "user" }); // use "user" to use front-cam on mobile phones

      // append elements to the DOM --> **before starting the webcam**
      if (this.isIos) {
        document
          .getElementById("webcam-container")
          .appendChild(this.webcam.webcam); // webcam object needs to be added in any case to make this work on iOS
        // grab video-object in any way you want and set the attributes --> **"muted" and "playsinline"**
        const wc = document.getElementsByTagName("video")[0];
        wc.setAttribute("playsinline", true); // written with "setAttribute" bc. iOS buggs otherwise :-)
        wc.muted = "true";
        wc.id = "webcamVideo";
        wc.style.width = width + "px";
        wc.style.height = height + "px";
      } else {
        document
          .getElementById("webcam-container")
          .appendChild(this.webcam.canvas);
      }
      // append elements to the dom
      this.labelContainer = document.getElementById("label-container");
      for (let i = 0; i < this.maxPredictions; i++) {
        // and class labels
        this.labelContainer.appendChild(document.createElement("div"));
      }
      this.resultContainer = document.getElementById("result-container");
      this.resultContainer.appendChild(document.createElement("div"));

      // only now start the webcam --> **after video-object added to DOM and attributes are set**
      this.webcam.play();
      window.requestAnimationFrame(this.loop); // update canvas by loop-function
    },
    async loop() {
      this.webcam.update(); // update the webcam frame
      await this.predict();
      window.requestAnimationFrame(this.loop);
    },
    // run the webcam image through the image model
    async predict() {
      // predict can take in an image, video or canvas html element
      let prediction;
      if (this.isIos) {
        prediction = await this.model.predict(this.webcam.webcam);
      } else {
        prediction = await this.model.predict(this.webcam.canvas);
      }

      for (let i = 0; i < this.maxPredictions; i++) {
        const classPrediction =
          prediction[i].className + ": " + prediction[i].probability.toFixed(2);
        this.labelContainer.childNodes[i].innerHTML = classPrediction;

        if (prediction[0].probability > prediction[1].probability) {
          this.resultContainer.childNodes[0].innerHTML = "쓰셨군요!";
        } else {
          this.resultContainer.childNodes[0].innerHTML = "안쓰셨군요!";
        }
      }
    },
  },
};
</script>
<style>
@import url("https://fonts.googleapis.com/css2?family=Poor+Story&display=swap");

* {
  margin: 0;
  padding: 0;
}

.template-box {
  min-width: 375px;
  width: 100vw;
  height: 100vh;

  display: flex;
  justify-content: center;
  align-items: center;

  background-color: #42b883;
}

.content-box {
  width: 320px;
  height: 600px;
  border-radius: 1rem;

  display: flex;
  align-items: center;
  flex-direction: column;

  background-color: #ffffff;
}

#title-container {
  padding: 2.5rem;
  font-size: 1.8rem;
  font-family: "Poor Story", cursive;
}

#webcam-container {
  overflow: hidden;
  text-align: center;
  padding: 0.5rem;
  width: 250px;
  height: 250px;
  border: 1px solid #35495e;
}

#webcam-container video {
  transform: rotateY(180deg);
  -webkit-transform: rotateY(180deg);
  -moz-transform: rotateY(180deg);

  object-fit: cover;
  width: 250px;
  height: 250px;
}

#label-container {
  padding: 2rem;
  font-size: 1.5rem;
  font-family: "Poor Story", cursive;
}
#result-container {
  font-size: 3rem;
  font-family: "Poor Story", cursive;
}
</style>
