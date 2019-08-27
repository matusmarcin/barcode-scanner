<template>
  <div class="barcode-scanner">
    Ahoj! 👋 Klikni ▶️ a ukáž EAN!
    <h1>
      <code>{{code ? code : '...'}}</code>
      <small v-if="code" v-clipboard="code" class="clipboard" @success="copied">📋</small>
    </h1>
    <div id="scanner">
      <!-- <div id="target">
        <div id="target-inner">
        </div>
      </div> -->
      <img id="screenshot">
    </div>
    <template v-if="isReady">
      <h2 @click="stop" v-if="isScanning">
        🛑
      </h2>
      <h2 @click="start" v-else-if="!code">
          ▶️
      </h2>
      <p v-else>
        <a href="#" @click.prevent="reloadPage">Urob to znovu! ⟳</a>
      </p>
    </template>
    <template v-else>
      <p><b>Povoľ kameru!</b></p>
    </template>
    <p><a href="#" @click.prevent="debugShown = !debugShown">Debug ⊞</a></p>
    <p v-if="debugShown" class="debug">
      Satisfactory count: <code>{{satisfactoryFoundCount}}</code><br />
      Codes found (and their count):<br />
      <pre><code>{{codesFound}}</code></pre>
    </p>  
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
      code: '',
      codesFound: {},
      satisfactoryFoundCount: 10,
      isReady: false,
      isScanning: false,
      debugShown: false,
      resultCollector: null,
    };
  },
  mounted() {
    /*
      TODO     
      [x] Use larger resolution
      [ ] Limit search area better
      [ ] Maybe 8 workers?
      [x] Collect barcodes, store in array with count and show the most frequent one
      [ ] Use image? Take a screenshot and process that.
      [ ] Re-starting probably needs init again? Video dies.

     */ 
    this.resultCollector = Quagga.ResultCollector.create({
        capture: true,
        capacity: 40,
        filter: function(codeResult) {
          // only store results which match this constraint
          // returns true/false
          // e.g.: return codeResult.format === "ean_13";
          return true;
        }
    });

    Quagga.init({
      inputStream: {
        name: "Live",
        type: "LiveStream",
        target: document.querySelector('#scanner'),    // Or '#yourElement' (optional)
        singleChannel: false,
        constraints: {
          width: 800,
          height: 600
        }
        // area: { // defines rectangle of the detection/localization area
        //   top: "20%",    // top offset
        //   right: "20%",  // right offset
        //   left: "20%",   // left offset
        //   bottom: "20%"  // bottom offset
        // }
      },
      // frequency: 50, // default is 10?
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
        Quagga.registerResultCollector(this.resultCollector);
        Quagga.onProcessed(this.maybeFound);
        Quagga.onDetected(this.found);
        this.ready();
    },
    found: function (data) {
      console.log(data);
      this.code = this.logCode(data.codeResult.code);
      if (this.isSatisfactory()) {
        // TODO Take a screenshot first and replace video
        this.screenshot();
        this.stop();
      }
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

    },
    logCode: function (code) {
      if (typeof this.codesFound[code] !== 'undefined') {
        this.codesFound[code]++;
      } else {
        this.codesFound[code] = 1;
      }
      return Object.keys(this.codesFound).reduce((mostFrequentCode, currentCode) => {
        const currentCodeCount = this.codesFound[currentCode];
        return this.codesFound[mostFrequentCode] < currentCodeCount ? currentCode : mostFrequentCode;
      }, code);
    },
    ready: function () {
      this.isReady = true;
    },
    start: function () {
      this.code = '';
      this.codesFound = {};
      Quagga.start();
      this.isScanning = true;
    },
    stop: function () {
      this.isScanning = false;
      Quagga.stop();
    },
    isSatisfactory: function () {
      return this.codesFound[this.code] >= this.satisfactoryFoundCount;
    },
    screenshot: function () {
      const canvas = Quagga.canvas.dom.image;
      document.getElementById('screenshot').setAttribute('src', canvas.toDataURL());
    },
    reloadPage: function () {
      window.location.reload();
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
  height: 240px;
  width: 320px;
  margin: 0 auto;
}
#scanner video,
#scanner canvas,
#scanner #target,
#scanner #screenshot {
  position: absolute;
  top: 0;
  left: 0;
  z-index: 1;
  width: 320px;
  height: 240px;
}
#scanner #screenshot {
  z-index: 3;
}
#scanner canvas {
  z-index: 4;
}
#scanner #target {
  padding: 18% 20% 20%;
  height: 288px;
  width: 384px;
  z-index: 5;
}
#scanner #target-inner {
  /* border: 2px solid red; */
  height: 284px;
  width: 380px;
  z-index: 4;
}
.debug {
  text-align: left;
  color: #fff;
  background: #333;
  padding: 2em;
}
h2 {
  font-size: 3rem;
  margin-top: 1rem;
  margin-bottom: 1rem;
}
</style>
