<template>
  <div class=" mix-auto d-flex justify-center align-center" style="height: 95vh">

    <v-card>
      <v-card-text>
        <!--Show QR Code Scanner -->
        <video class="qr-video"></video>

        <!-- Start Scanner -->
         <v-btn class="mb-5" block color="light blue" @click="startScanner">Start Scanner</v-btn>

         <!-- Stop Scanner -->
          <v-btn block color="error">Stop Scanner</v-btn>
      </v-card-text>
    </v-card>
    
  </div>
</template>

<script lang="ts" setup>
//@ts-nocheck
import QrScanner from 'qr-scanner'

const videoRef = ref<HTMLVideoElement | null>(null)
// const result = ref('')
let scanner: QrScanner | null=null

const startScanner = async() => {
  if(!videoRef.value) return
  scanner = new QrScanner(
    videoRef.value,
  (scanResult) => {
    //result.value = scanResult.data
    // emit('scanned. scanResult.data')

    //optional: stop After successful scan
     scanner?.stop()

  },
  {
    preferredCamera: 'environment',
    highlightScanRegion: true,
    highlightCodeOutline: true,

  }
)
}
</script>

<style>
  .qr-video{
    width: 100%;
    height: 400px;
    border-radius: 12pz;
    background: #000;
  }
</style>