<template>
  <div ref="videoWidget" class="video-widget">
    <statsForNerds v-if="widget.options.statsForNerds" :stream-name="externalStreamId" />
    <div v-if="nameSelectedStream === undefined" class="no-video-alert">
      <span>No video stream selected.</span>
    </div>
    <div
      v-else-if="!namesAvailableStreams.isEmpty() && !namesAvailableStreams.includes(nameSelectedStream)"
      class="no-video-alert"
    >
      <p>The selected stream "{{ nameSelectedStream }}" is not available.</p>
      <p>Available ones are: {{ namesAvailableStreams.map((name) => `"${name}"`).join(', ') }}.</p>
      <br />
      <p>
        This can happen if you changed vehicles and the stream name in the new one is different from the former, or if
        the source is not available at all.
      </p>
      <br />
      <p>
        Please open this video player configuration and select a new stream from the ones available, or check your
        source for issues.
      </p>
    </div>
    <div v-else-if="!streamConnected" class="no-video-alert">
      <div class="no-video-alert">
        <p>
          <span class="text-xl font-bold">Server status: </span>
          <span v-for="(statusParagraph, i) in serverStatus.toString().split('\\n')" :key="i">
            {{ statusParagraph }}
            <br />
          </span>
        </p>
        <p>
          <span class="text-xl font-bold">Stream status: </span>
          <span v-for="(statusParagraph, i) in streamStatus.toString().split('\\n')" :key="i">
            {{ statusParagraph }}
            <br />
          </span>
        </p>
      </div>
    </div>
    <div v-else class="no-video-alert">
      <p>Loading stream...</p>
    </div>
    <div>
      <canvas ref="canvas"></canvas>
      <video id="videoElements" muted autoplay playsinline disablePictureInPicture loop :style="{ filter: combinedFilters }" >
        Your browser does not support the video tag.
        <source src="/home/j12968/blueos/my-cockpit/src/components/widgets/test.mp4" type="video/mp4">
      </video>
      <!-- <video id="mainDisplayStream" ref="videoElement" muted autoplay playsinline disablePictureInPicture :style="{ filter: combinedFilters }" >
        Your browser does not support the video tag.
      </video> -->
    </div>
  </div>
  <v-dialog v-model="widgetStore.widgetManagerVars(widget.hash).configMenuOpen" width="auto">
    <v-card class="pa-4 text-white" style="border-radius: 15px" :style="interfaceStore.globalGlassMenuStyles">
      <v-card-title class="text-center">Video widget config</v-card-title>
      <v-card-text class="flex flex-col gap-y-4">
        <v-select
          v-model="nameSelectedStream"
          label="Stream name"
          class="my-3"
          :items="namesAvailableStreams"
          item-title="name"
          density="compact"
          variant="outlined"
          no-data-text="No streams available."
          hide-details
          return-object
        />
        <v-select
          v-model="widget.options.videoFitStyle"
          label="Fit style"
          class="my-3"
          :items="['cover', 'fill', 'contain']"
          item-title="style"
          density="compact"
          variant="outlined"
          no-data-text="No streams available."
          hide-details
          return-object
        />
        <v-banner-text>Saved stream name: "{{ widget.options.internalStreamName }}"</v-banner-text>
        <v-switch
          v-model="widget.options.flipHorizontally"
          class="my-1"
          label="Flip horizontally"
          :color="widget.options.flipHorizontally ? 'white' : undefined"
          hide-details
        />
        <v-switch
          v-model="widget.options.flipVertically"
          class="my-1"
          label="Flip vertically"
          :color="widget.options.flipVertically ? 'white' : undefined"
          hide-details
        />
        <v-switch
          v-model="widget.options.statsForNerds"
          class="my-1"
          label="Stats for nerds"
          :color="widget.options.statsForNerds ? 'white' : undefined"
          hide-details
        />
        <v-switch
          v-model="widget.options.isInvertFilterOn"
          class="my-1"
          label="Invert"
          :color="widget.options.isInvertFilterOn ? 'white' : undefined"
          hide-details
        />
        <v-slider
          v-model="widget.options.invertRate"
          label="Invert Rate"
          color="white"
          :min="0"
          :max="100"
          thumb-label
          :step="1"
        />
        <v-switch
          v-model="widget.options.isSaturateFilterOn"
          class="my-1"
          label="Saturate"
          :color="widget.options.isSaturateFilterOn ? 'white' : undefined"
          hide-details
        />
        <v-slider
          v-model="widget.options.saturateRate"
          label="Saturate Rate"
          color="white"
          :min="0"
          :max="1000"
          thumb-label
          :step="1"
        />
        <v-switch
          v-model="widget.options.isBrightnessFilterOn"
          class="my-1"
          label="Brightness"
          :color="widget.options.isBrightnessFilterOn ? 'white' : undefined"
          hide-details
        />
        <v-slider
          v-model="widget.options.brightnessRate"
          label="Brightness Rate"
          color="white"
          :min="0"
          :max="1000"
          thumb-label
          :step="1"
        />
        <v-switch
          v-model="widget.options.isContrastFilterOn"
          class="my-1"
          label="Contrast"
          :color="widget.options.isContrastFilterOn ? 'white' : undefined"
          hide-details
        />
        <v-slider
          v-model="widget.options.contrastRate"
          label="Contrast Rate"
          color="white"
          :min="0"
          :max="1000"
          thumb-label
          :step="1"
        />
        <v-switch
          v-model="widget.options.ishueRotateFilterOn"
          class="my-1"
          label="HueRotate"
          :color="widget.options.ishueRotateFilterOn ? 'white' : undefined"
          hide-details
        />
        <v-slider
          v-model="widget.options.hueDeg"
          label="Hue Deg"
          color="white"
          :min="0"
          :max="360"
          thumb-label
          :step="1"
        />
        <div class="flex-wrap justify-center d-flex ga-5">
          <v-btn prepend-icon="mdi-file-rotate-left" variant="outlined" @click="rotateVideo(-90)"> Rotate Left</v-btn>
          <v-btn prepend-icon="mdi-file-rotate-right" variant="outlined" @click="rotateVideo(+90)"> Rotate Right</v-btn>
        </div>
      </v-card-text>
    </v-card>
  </v-dialog>
</template>

<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { computed, onMounted, onBeforeMount, onBeforeUnmount, ref, toRefs, watch } from 'vue'

import StatsForNerds from '@/components/VideoPlayerStatsForNerds.vue'
import { isEqual } from '@/libs/utils'
import { useAppInterfaceStore } from '@/stores/appInterface'
import { useVideoStore } from '@/stores/video'
import { useWidgetManagerStore } from '@/stores/widgetManager'
import type { Widget } from '@/types/widgets'
const interfaceStore = useAppInterfaceStore()

const videoStore = useVideoStore()
const widgetStore = useWidgetManagerStore()

const { namessAvailableAbstractedStreams: namesAvailableStreams } = storeToRefs(videoStore)

const props = defineProps<{
  /**
   * Widget reference
   */
  widget: Widget,
  isFilterOn: boolean
}>()

const widget = toRefs(props).widget

const nameSelectedStream = ref<string | undefined>()
const videoElement = ref<HTMLVideoElement | undefined>()
const mediaStream = ref<MediaStream | undefined>()
const streamConnected = ref(false)

onBeforeMount(() => {
  // Set the default initial values that are not present in the widget options
  const defaultOptions = {
    videoFitStyle: 'cover',
    flipHorizontally: false,
    flipVertically: false,
    rotationAngle: 0,
    statsForNerds: false,
    internalStreamName: undefined as string | undefined,
    isInvertFilterOn: true,
    invertRate: 100,
    isSaturateFilterOn: true,
    saturateRate: 462,
    isBrightnessFilterOn: false,
    brightnessRate: 100,
    isContrastFilterOn: false,
    contrastRate: 100,
    ishueRotateFilterOn: true,
    hueDeg: 82
  }
  widget.value.options = Object.assign({}, defaultOptions, widget.value.options)
  nameSelectedStream.value = widget.value.options.internalStreamName
})

import * as cv from "@techstark/opencv-js";
const canvas = ref<HTMLCanvasElement | null>(null);

onMounted(() => {
  console.log("opencv loaded");
  console.log(cv);

  const video = document.getElementById('videoElements') as HTMLVideoElement;
  const ctx = canvas.value?.getContext('2d');

  video.addEventListener('play', () => {
    setInterval(() => {
      if (ctx) {
        ctx.drawImage(video, 0, 0, canvas.value!.width, canvas.value!.height);
        const imageData = ctx.getImageData(0, 0, canvas.value!.width, canvas.value!.height);
        // 画像処理を行う
        const pixels = imageData.data;
        for (let i = 0; i < pixels.length; i += 4) {
          pixels[i] = 255 - pixels[i]; // 赤色を反転
          pixels[i + 1] = 255 - pixels[i + 1]; // 緑色を反転
          pixels[i + 2] = 255 - pixels[i + 2]; // 青色を反転
        }
        ctx.putImageData(imageData, 0, 0);
      }
    }, 16); // 16msごとに処理を行う
  });
});

const combinedFilters = computed(() => {
  let filters = [];

  if (widget.value.options.isInvertFilterOn) {
    filters.push(`invert(${widget.value.options.invertRate / 100})`);
  }
  if (widget.value.options.isSaturateFilterOn) {
    filters.push(`saturate(${widget.value.options.saturateRate / 100})`);
  }
  if (widget.value.options.isBrightnessFilterOn) {
    filters.push(`brightness(${widget.value.options.brightnessRate / 100})`);
  }
  if (widget.value.options.isContrastFilterOn) {
    filters.push(`contrast(${widget.value.options.contrastRate / 100})`);
  }
  if (widget.value.options.ishueRotateFilterOn) {
    filters.push(`hue-rotate(${widget.value.options.hueDeg}deg)`);
  }

  return filters.length ? filters.join(' ') : 'none'; // フィルタをスペースで区切って適用
});

const externalStreamId = computed(() => {
  return nameSelectedStream.value ? videoStore.externalStreamId(nameSelectedStream.value) : undefined
})

watch(
  () => videoStore.streamsCorrespondency,
  () => {
    mediaStream.value = undefined

    if (!nameSelectedStream.value) return

    const selectedExternalId = videoStore.externalStreamId(nameSelectedStream.value)
    if (!selectedExternalId) return

    const newStreamCorr = videoStore.streamsCorrespondency.find((stream) => stream.externalId === selectedExternalId)
    if (!newStreamCorr) return

    const newInternalName = newStreamCorr.name

    if (nameSelectedStream.value !== newInternalName) {
      nameSelectedStream.value = newInternalName
      widget.value.options.internalStreamName = newInternalName
    }
  },
  { deep: true }
)

const streamConnectionRoutine = setInterval(() => {
  // If the video player widget is cold booted, assign the first stream to it
  if (widget.value.options.internalStreamName === undefined && !namesAvailableStreams.value.isEmpty()) {
    widget.value.options.internalStreamName = namesAvailableStreams.value[0]
    nameSelectedStream.value = widget.value.options.internalStreamName
  }

  if (externalStreamId.value !== undefined) {
    const updatedMediaStream = videoStore.getMediaStream(externalStreamId.value)
    // If the widget is not connected to the MediaStream, try to connect it
    if (!isEqual(updatedMediaStream, mediaStream.value)) {
      mediaStream.value = updatedMediaStream
    }

    const updatedStreamState = videoStore.getStreamData(externalStreamId.value)?.connected ?? false
    if (updatedStreamState !== streamConnected.value) {
      streamConnected.value = updatedStreamState
    }
  }

  if (!namesAvailableStreams.value.isEmpty() && !namesAvailableStreams.value.includes(nameSelectedStream.value!)) {
    if (videoStore.lastRenamedStreamName !== '') {
      nameSelectedStream.value = videoStore.lastRenamedStreamName
      return
    }
    nameSelectedStream.value = namesAvailableStreams.value[0]
  }
}, 1000)
onBeforeUnmount(() => clearInterval(streamConnectionRoutine))

watch(nameSelectedStream, () => {
  widget.value.options.internalStreamName = nameSelectedStream.value
  mediaStream.value = undefined
})

watch(mediaStream, () => {
  if (!videoElement.value || !mediaStream.value) return
  videoElement.value.srcObject = mediaStream.value
  videoElement.value
    .play()
    .then(() => console.log('[VideoPlayer] Stream is playing'))
    .catch((reason) => {
      const msg = `Failed to play stream. Reason: ${reason}`
      console.error(`[VideoPlayer] ${msg}`)
    })
})

const rotateVideo = (angle: number): void => {
  widget.value.options.rotationAngle += angle
}

const flipStyle = computed(() => {
  return `scale(${widget.value.options.flipHorizontally ? -1 : 1}, ${widget.value.options.flipVertically ? -1 : 1})`
})

const rotateStyle = computed(() => {
  return `rotate(${widget.value.options.rotationAngle ?? 0}deg)`
})

const transformStyle = computed(() => {
  return `${flipStyle.value} ${rotateStyle.value}`
})

const serverStatus = computed(() => {
  if (externalStreamId.value === undefined) return 'Unknown.'
  return videoStore.getStreamData(externalStreamId.value)?.webRtcManager.signallerStatus ?? 'Unknown.'
})

const streamStatus = computed(() => {
  if (externalStreamId.value === undefined) return 'Unknown.'

  const availableSources = videoStore.availableIceIps
  if (!availableSources.isEmpty() && !availableSources.find((ip) => videoStore.allowedIceIps.includes(ip))) {
    return `Stream is coming from IPs [${availableSources.join(', ')}], which are not in the list of allowed sources
      [${videoStore.allowedIceIps.join(', ')}].\\n Please check your configuration.`
  }
  return videoStore.getStreamData(externalStreamId.value)?.webRtcManager.streamStatus ?? 'Unknown.'
})
</script>

<style scoped>
.video-widget {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
video {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  object-fit: v-bind('widget.options.videoFitStyle');
  transform: v-bind('transformStyle');
  z-index: -1;
}
.no-video-alert {
  width: 100%;
  height: 100%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: rgb(0, 20, 60);
  text-align: center;
  vertical-align: middle;
  padding: 3rem;
  color: white;
  border: 2px solid rgb(0, 20, 80);
}
canvas {
  height: 100%;
  width: 100%;
  position: absolute;
  top: 0;
  left: 0;
  object-fit: v-bind('widget.options.videoFitStyle');
  transform: v-bind('transformStyle');
}
</style>
