<template>
  <div class="scanner-container">
    <h2>Scanner QR Code & Barcode</h2>

    <div>
      <button @click="StartScanQrCode">Start Scan</button>
      <button @click="StartBarcodeScan">Start Barcode Scan</button>
    </div>
    <!-- SCANNER LIVE -->
    <div v-if="cameraSupported">
      <div id="reader" class="reader-box"></div>
      <div class="below-reader">
        <br />
      </div>
    </div>

    <div v-else class="error-message">
      Il tuo dispositivo non supporta la scansione tramite fotocamera.
    </div>

    <div v-if="scanResult" class="result-box">
      Codice letto: {{ scanResult }}
    </div>
  </div>
</template>

<script lang="ts" setup>
import { ref, onMounted, onBeforeUnmount } from "vue";
import { Html5QrcodeScanner, Html5QrcodeSupportedFormats } from "html5-qrcode";
import { BrowserMultiFormatReader } from "@zxing/browser";

const scanResult = ref<string | null>(null);
const cameraSupported = ref(true);

const scannerId = "reader";
let scanner: Html5QrcodeScanner | null = null;

// SCANNER
onMounted(() => {
  // Check if camera is supported 
  if (!navigator.mediaDevices?.getUserMedia) {
    cameraSupported.value = false;
    return;
  }
});

// Stop scanner when the component is unmounted
onBeforeUnmount(() => {
  scanner?.clear().catch((err) => {
    console.error("Errore nello stop dello scanner:", err);
  });
});

const StartScanQrCode = () => {
  scanner = new Html5QrcodeScanner(
    scannerId,
    {
      fps: 10,
      aspectRatio: 1.0,
      disableFlip: false,
      qrbox: { width: 200, height: 200 },
      videoConstraints: {
        facingMode: "environment",
        //high to improve the quality
        width: { ideal: 3920 },
        height: { ideal: 2080 },
        advanced: [{ focusMode: "continuous" }] as any[],
      },
      formatsToSupport: [
        Html5QrcodeSupportedFormats.QR_CODE,
      ],
    },
    false
  );

  // Success callback
  scanner.render(
    (decodedText) => {
      scanResult.value = decodedText;
      console.log("Codice letto (live):", decodedText);
    },
    (errorMessage) => {
      console.debug("Errore lettura:", errorMessage);
    }
  );
};
const StartBarcodeScan = () => {
  
  scanner = new Html5QrcodeScanner(
    scannerId,
    {
      fps: 10,
      aspectRatio: 1.0,
      disableFlip: false,
      qrbox: {width: 300, height: 100},
      videoConstraints: {
        facingMode: "environment",
        //high to improve the quality
        width: { ideal: 3920 },
        height: { ideal: 2080 },
        advanced: [{ focusMode: "continuous" }] as any[],
      },
      formatsToSupport: [
        Html5QrcodeSupportedFormats.CODE_39,
        Html5QrcodeSupportedFormats.CODE_128,
        Html5QrcodeSupportedFormats.EAN_13,
        Html5QrcodeSupportedFormats.EAN_8,
        Html5QrcodeSupportedFormats.UPC_A,
        Html5QrcodeSupportedFormats.UPC_E,
        Html5QrcodeSupportedFormats.ITF,
        Html5QrcodeSupportedFormats.CODABAR,
      ],
    },
    false
  );

  // Success callback
  scanner.render(
    (decodedText) => {
      scanResult.value = decodedText;
      console.log("Codice letto (live):", decodedText);
    },
    (errorMessage) => {
      console.debug("Errore lettura:", errorMessage);
    }
  );
};
</script>

<style scoped>
.scanner-container {
  padding: 1rem;
  max-width: 360px;
  margin: 0 auto;
  text-align: center;
  color: white;
  background: #000000;
  border-radius: 8px;
}

.reader-box {
  width: 100%;
  max-width: 480px;
  height: auto;
  min-height: 400px;
  margin: 0 auto;
  border: none; /* opzionale */
  background: none;
}

.below-reader {
  margin-top: 1rem;
  color: #fff;
  background: rgba(75, 11, 11, 0.5);
  padding: 0.5rem 1rem;
  border-radius: 4px;
  font-size: 0.9rem;
}

.result-box {
  margin-top: 1rem;
  font-weight: bold;
  color: lightgreen;
  word-break: break-word;
}

.upload-box {
  margin-top: 1rem;
}

.error-message {
  margin-top: 1rem;
  color: #ff4444;
  font-weight: bold;
}
</style>
