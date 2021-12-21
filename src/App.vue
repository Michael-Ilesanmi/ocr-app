<template>
  <div id="app" class="col-10 m-auto my-5 d-flex flex-column align-items-center">
        <div class="col-8">
            <div id="uploadArea" class="col-12 d-flex my-3 flex-column align-items-center justify-content-center" v-on:click="clickUp">
                <input type="file" name="cert-file" ref="file" accept=".png, .jpeg, .jpg" id="cert-file" v-on:change="recognize" hidden>
                <div class="">
                    <i class="fas fa-cloud-upload-alt"></i>
                    <p class="">Browse file to Upload</p>
                </div>
            </div>
            <div id="myProgress" ref='myProgress' class="col-12" style="display:none">
              <!-- <div id="myBar"></div> -->
            </div>
        </div>
        <div class="col-8 d-flex flex-md-row flex-column my-md-5 justify-content-between">
            <div id="preview" ref="preview" class="my-md-0 my-3 col-md-5 col-12 d-flex flex-column align-items-center justify-content-center">
                  <!-- Image Preview will show here -->
                  <img id='image-preview' ref="imageReview" src="">
            </div>
            <div  id="result-r" class="my-md-0 my-3 col-md-5 col-12 d-flex flex-column align-items-center justify-content-center">
              <div class="px-4 py-5" v-on:click='clipboardCopy'> 
                  <!-- Result will show here -->
                  <p v-if="!data.text">
                    Your result will show here!!!
                  </p>
                  <p id="result" ref="result"></p>
              </div>
            </div>
        </div>
        <div id="confidenceLevel" class="col-12" style="" v-if='data.confidence'>
          <!-- The programm returns a confidence level of the result -->
            <div id="myBar" :style="`width:${data.confidence}%;background-color:${data.confidenceColor}`"><b>Confidence Level :  {{data.confidence}} %</b></div>;
        </div>
  </div>
</template>

<script>
/* eslint-disable */
import { createWorker, PSM, OEM } from 'tesseract.js';
const worker = createWorker({
  logger: m => {
        var progressPer = Math.ceil(m.progress*100);
          if (m.status=='recognizing text') {
            var myBar = `<div id="myBar" style="width:${progressPer}%"><b>Analyzing : ${progressPer}%</b></div>`;
            var progressArea = document.querySelector('#myProgress');
            progressArea.style.display='block'
            progressArea.innerHTML=myBar;
          }else if (m.status!='recognizing text') {
            var myBar = `<div id="myBar" style="width:0.1%;"></div>`;
            var progressArea = document.querySelector('#myProgress');
            progressArea.innerHTML=myBar;
          }
        }
  });
export default {
  name: 'app',
  data(){
    return{
        file:'',
        data:{
          text:null,
          confidence:'',
          confidenceColor:'',
        }
      }
  },
  methods: {
        async recognize(){
            var file = this.$refs.file.files[0];
            if (file) {
            const img = this.$refs.imageReview;
            img.classList.add("obj");
            img.file = this.file;
            var preview = this.$refs.preview;
            preview.appendChild(img);
            const reader = new FileReader();
            reader.onload = (function(aImg) { return function(e) { aImg.src = e.target.result; }; })(img);
            reader.readAsDataURL(file);
            //
            const image = document.querySelector('.obj');
            await worker.load();
            await worker.loadLanguage('eng');
            await worker.initialize('eng', OEM.LSTM_ONLY);
            await worker.setParameters({
              tessedit_pageseg_mode: PSM.SINGLE_BLOCK,
            });
            const { data: { text, confidence } } = await worker.recognize(image);
            this.$refs.result.innerHTML=text;
            //
            //this part just some minor customization
            if (confidence>90) {
                  this.data.confidenceColor = '#48c800';
            }
            if (confidence<90 && confidence > 65) {
                  this.data.confidenceColor = '#8000ff';
            }
            if (confidence<65 && confidence > 50) {
                  this.data.confidenceColor = '#FFF52E';
            }
            if (confidence<50) {
                  this.data.confidenceColor = '#FF2F13';
            }
            this.data.confidence = confidence;
            this.data.text= text;
          }else{
            console.log('empty')
          }
        },
        clickUp(){
            document.querySelector('#cert-file').click();
        },
        //to copy tthe result to clipboard
        clipboardCopy() {
            let text = document.getElementById("result").textContent;
            const el = document.createElement("textarea");
            el.value = text;
            document.body.appendChild(el);
            el.select();
            document.execCommand("copy");
            document.body.removeChild(el);
            if (document.execCommand("copy")) {
                  var tooltip = document.getElementById("myTooltip");
                  tooltip.innerHTML = "Copied to clipboard ";                
            }
        },
    },
  }
</script>

<style>
  .obj, #result{
    max-width: 100%;
    max-height: 100%;
  }
  #result{
    font-size: 0.7rem;
    /* height: 100px; */
    width: 100%;
    font-weight: 600;
  }
#uploadArea, #preview, #result-r{
  height: 350px;
  border: 3px dashed #6990f2;
  border-radius: 8px;
  color: #6990f2;
  font-weight: 500;
}
#uploadArea p{
  font-size: 1.5rem;
  font-weight: 600;
}
#uploadArea i{
  font-size: 5.5rem;
}
#myBar >b{
  position:absolute;
  left:0;
  right:0;
  color: #fff;
}
#myProgress, #confidenceLevel {
  width: 100%;
  background-color: grey;
  border-radius: 10px;
  height: 29px;
  overflow: hidden;
}

#myBar {
  height: 30px;
  background-color: #6990f2;
}

#app {
  font-family: 'Montserrat', sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
  margin-top: 60px;
}

</style>
