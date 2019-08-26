<template>
  <div class="barcode-scanner">
    Ukáž EAN!
    <h1>
      <code>{{code ? code : '...'}}</code>
      <small v-if="code" v-clipboard="code" class="clipboard" @success="copied">📋</small>
    </h1>
    <div id="scanner"></div>
  </div>
</template>

<script>
import Quagga from 'quagga';

export default {
  name: 'BarcodeScanner',
  props: {    
  },
  data() {
    return {
      code: ''
    }
  },
  mounted() {
    Quagga.init({
      inputStream: {
        name: "Live",
        type: "LiveStream",
        target: document.querySelector('#scanner')    // Or '#yourElement' (optional)
      },
      decoder: {
        readers : ["ean_reader"]
      },
      locator: {
        halfSample: true,
        patchSize: "medium", // x-small, small, medium, large, x-large
      }
    }, this.onInit);
  },
  methods: {
    onInit: function (err) {
        if (err) {
            console.log(err);
            return
        }
        console.log("Initialization finished. Ready to start");
        Quagga.onProcessed(this.maybeFound);
        Quagga.onDetected(this.found);        
        Quagga.start();
    },
    found: function (data) {
      console.log(data);
      this.code = data.codeResult.code;
    },
    maybeFound: function (result) {
        var drawingCtx = Quagga.canvas.ctx.overlay,
            drawingCanvas = Quagga.canvas.dom.overlay;
            // console.log(drawingCanvas);

        if (result) {
            if (result.boxes) {
                drawingCtx.clearRect(0, 0, parseInt(drawingCanvas.getAttribute("width")), parseInt(drawingCanvas.getAttribute("height")));
                result.boxes.filter(function (box) {
                    return box !== result.box;
                }).forEach(function (box) {
                    Quagga.ImageDebug.drawPath(box, {x: 0, y: 1}, drawingCtx, {color: "green", lineWidth: 2});
                });
            }

            if (result.box) {
                Quagga.ImageDebug.drawPath(result.box, {x: 0, y: 1}, drawingCtx, {color: "#00F", lineWidth: 2});
            }

            if (result.codeResult && result.codeResult.code) {
                Quagga.ImageDebug.drawPath(result.line, {x: 'x', y: 'y'}, drawingCtx, {color: 'red', lineWidth: 3});
            }
        }
    },
    copied: function () {

    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style>
.clipboard {
  color: #ccc;
  font-size: 1rem;
  display: inline-block;
  margin: 0.2rem 0.5rem 0.1rem 0.5rem;
}
#scanner {
  position: relative;
  max-height: 480px;
  max-width: 640px;
  margin: 0 auto;
}
#scanner video,
#scanner canvas {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
}
#scanner canvas {
  z-index: 2;
}
</style>
