<script setup lang="ts">
import { ref } from 'vue'
import FilterPanelCrop from './components/FilterPanel/Crop.vue'

interface FilterVideoStream {
  key: string
  title: string
  panel: any
  args: {
    [key: string]: any
  }
  filter: (
    args: {
      [key: string]: any
    },
    canvas: HTMLCanvasElement,
    canvasCtx: CanvasRenderingContext2D,
    track: MediaStreamTrack,
    proc: any,
    reader: any,
    frame: any
  ) => Promise<void>
}

const manageFilterWindowShow = ref(false)
// const pageState = ref<'preview'|'manage_filter'>('preview')
// const playerState = ref<'stop'|'play'>('stop')

const previewVideo = ref<HTMLVideoElement>()
const previewMediaStream = ref<MediaStream>()

const filters = ref<FilterVideoStream[]>([
  {
    key: 'crop',
    panel: FilterPanelCrop,
    title: 'Crop',
    args: {
      w: 0,
      h: 0,
      x: 0,
      y: 0,
      originalW: 0,
      originalH: 0,
    },
    filter: async (args, cvs, ctx, _track, _proc, _reader, _frame) => {
      // get current full frame
      const currentFrame = ctx.getImageData(0, 0, cvs.width, cvs.height)
      if (args.w === 0 && args.h === 0 && args.originalW === 0 && args.originalH === 0) {
        args.originalW = currentFrame.width
        args.originalH = currentFrame.height
        args.w = currentFrame.width
        args.h = currentFrame.height
        return
      }
      // console.log(args, cvs, ctx, track, proc, reader, value)
      // clear ctx
      ctx.clearRect(0, 0, cvs.width, cvs.height)
      // crop canvas to options
      cvs.width = args.w
      cvs.height = args.h
      // draw cropped frame
      ctx.putImageData(currentFrame, args.x, args.y)
    }
  }
])

const start = async () => {
  const media = await navigator.mediaDevices.getDisplayMedia({
    video: true
  })
  const tracks = media.getTracks()

  // check supported
  if (!media) return alert ('media not found')
  if (tracks.length < 1) return alert('media track not found')
  if (!(window as any).MediaStreamTrackProcessor) return alert('browser not support MediaStreamTrackProcessor api')
  if (!previewVideo.value) return alert('web not fully loaded')

  // @ts-ignore
  const MediaStreamTrackProcessor = (window as any).MediaStreamTrackProcessor

  if (previewVideo.value) {
    let mediaStreamResult: MediaStream = new MediaStream()

    // tracks
    for (const track of tracks) {
      if (track.kind === 'video') {
        const cvs = document.createElement('canvas') as HTMLCanvasElement
        // const cvs = previewCanvas.value
        const ctx = cvs.getContext('2d')
        if (!ctx) return alert('browser 2d context not found')

        const proc = new MediaStreamTrackProcessor(track)
        const reader = proc.readable.getReader()
        const readChunk = async () => {
          const { done, value } = await reader.read()
          
          // the MediaStream video can have dynamic size
          if(cvs.width !== value.displayWidth || cvs.height !== value.displayHeight) {
            cvs.width = value.displayWidth
            cvs.height = value.displayHeight
          }
          ctx.clearRect(0, 0, cvs.width, cvs.height)
          // value is a VideoFrame
          ctx.drawImage(value, 0, 0)
          value.close() // close the VideoFrame when we're done with it\

          // run filters
          for (const filter of filters.value) {
            await filter.filter(filter.args, cvs, ctx, track, proc, reader, value)
          }

          // done
          if(!done) readChunk()
        }
        readChunk()
        
        // js
        mediaStreamResult = cvs.captureStream()
      }
    }

    const prv = previewVideo.value
    prv.srcObject = mediaStreamResult
    previewMediaStream.value = mediaStreamResult
    // ;(prv as any).captureStream = (prv as any).captureStream || (prv as any).mozCaptureStream
  }
}

const mngfilter = async () => {
  manageFilterWindowShow.value = !manageFilterWindowShow.value
}

const updateFilterArgs = (key: string, args: any) => {
  const filter = filters.value.find(f => f.key === key)
  if (filter) {
    filter.args = args
  }
}
</script>

<template>
  <div class="w-screen h-screen flex bg-slate-800 text-gray-200">
    <div class="flex-1 relative bg-black">
      <div class="flex space-x-2 fixed top-0 left-0 z-10">
        <button @click="start">start</button>
        <button @click="mngfilter">manage filters</button>
      </div>
      <video ref="previewVideo" id="preview" autoplay muted class="absolute w-full h-full bg-cover overflow-hidden border-2 border-blue-500" />
    </div>
    <div v-if="manageFilterWindowShow" class="flex-1 relative bg-slate-800 max-h-full overflow-y-auto">
      <div class="px-4 py-4 flex flex-col">
        <component v-for="filter in filters" :key="filter.key" :is="filter.panel" :args="filter.args" @update-args="(newArgs: any) => updateFilterArgs(filter.key, newArgs)" />
      </div>
    </div>
  </div>
</template>