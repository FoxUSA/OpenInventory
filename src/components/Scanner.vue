<template>
<v-dialog v-model="toggle" width="640">
  <v-card>
    <v-card-title class="headline">UPC Scanner</v-card-title>
    <v-card-actions v-if="toggle">
      <div id="scanner" class="ma-auto">
      </div>
    </v-card-actions>
  </v-card>
</v-dialog>
</template>

<script>
import Quagga from 'quagga'

const RENDER_TIMEOUT = 100

export default {
  props: {// <Scanner :scanner.sync="scanner" :callback="search" />
    callback: Function,
    scanner: Boolean
  },
  data: () => ({
    toggle: false
  }),
  watch: {
    '$route' (to, from) {
      try {
        Quagga.stop()
        this.toggle = false
      } catch (error) {}
    },
    scanner () { // SO much better than 2 way data binding #Sarcasm
      this.toggle = this.scanner
    },
    toggle (to, from) {
      this.$emit('update:scanner', this.toggle)
      if (!to) { return Quagga.stop() }

      setTimeout(() => {
        try {
          Quagga.init({
            inputStream: {
              name: 'Live',
              type: 'LiveStream',
              constraints: {
                width: 480,
                height: 480,
                facingMode: 'environment'
              },
              target: document.querySelector('#scanner') // Or '#yourElement' (optional)
            },
            locate: true,
            locator: {
              halfSample: true,
              patchSize: 'medium' // x-small, small, medium, large, x-large
            },
            decoder: {
              readers: ['upc_reader']
            }
          }, (err) => {
            if (err) {
              console.log(err)
              return
            }
            Quagga.start()
            Quagga.onDetected((result) => {
              Quagga.offDetected()
              Quagga.stop()

              this.callback(result.codeResult.code)
            })
          })
        } catch (error) {
          this.$toasted.error(`Error ${error}`)
        }
      }, RENDER_TIMEOUT)
    }
  },
  methods: {
    handleInput (e) {
      this.$emit('input', this.scanner)
    }
  }
}
</script>

<style>
#scanner {
  position: relative;
}

#scanner>canvas,
#scanner>video {
  max-width: 100%;
  width: 100%;
}

canvas.drawing,
canvas.drawingBuffer {
  position: absolute;
  left: 0;
  top: 0;
}
</style>
