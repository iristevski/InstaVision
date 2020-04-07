<template>
  <div id="app">
    <h1>Instagram Tags</h1>
    <br>
    <br>
    <picture-input
      ref="pictureInput"
      @change="onChange"
      width="600"
      height="600"
      margin="16"
      accept="image/jpeg, image/png"
      size="10"
      :removable="true"
      :customStrings="{
        drag: 'Drag image'
      }"
    ></picture-input>
    <br>
    <br>
    <p :data="hashtags">{{ hashtags }}</p>
    <!-- <a
      href="https://github.com/alessiomaffeis/vue-picture-input"
      class="btn btn-success" 
    >View project on GitHub</a>-->
  </div>
</template>

<script>
import PictureInput from "vue-picture-input";
import axios from "axios";
import { config } from "./config";

export default {
  name: "app",
  data() {
    return {
      gCloudVisionUrl:
        "https://vision.googleapis.com/v1/images:annotate?key=" + config.apiKey,
      hashtags: ""
    };
  },
  components: {
    PictureInput
  },
  methods: {
    async predictImage() {
      if (this.$refs.pictureInput.image) {
        let image = this.$refs.pictureInput.image.replace(
          /^data:image\/(png|jpg|jpeg);base64,/,
          ""
        );
        //console.log(image);
        let requestBody = {
          requests: [
            {
              image: {
                content: image
              },
              features: [
                {
                  type: "LABEL_DETECTION",
                  maxResults: 20
                }
              ]
            }
          ]
        };
        let predictionResults = await axios.post(
          this.gCloudVisionUrl,
          requestBody
        );
        let predictionResponse = predictionResults.data.responses[0];
        let annotations = predictionResponse.labelAnnotations;
        let allLabelDescriptions = annotations.map(annotation =>
          annotation.description.toLowerCase()
        );
        this.hashtags = "";
        for (var i = 0; i !== allLabelDescriptions.length; ++i) {
          this.hashtags +=
            "#" + allLabelDescriptions[i].replace(/\s+/g, "") + " ";
        }
        console.log(allLabelDescriptions);
      }
    },
    onChange() {
      console.log("New picture selected!");
      this.predictImage();
      if (this.$refs.pictureInput.image) {
        //console.log(this.$refs.pictureInput.image);
      } else {
        console.log("FileReader API not supported: use the <form>, Luke!");
      }
    }
  }
};
</script>

<style>
#app {
  font-family: "Avenir", Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

h1,
h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}
</style>
