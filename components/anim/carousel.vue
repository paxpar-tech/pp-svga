<script setup  lang="ts">
/*

Full screen feature, see https://www.techinfected.net/2020/05/fullscreen-toggle-component-in-vuejs.html
*/

import { ref, reactive, onMounted } from 'vue'

export interface CarouselConf {
  slides: Array<string>,
  background?: string,
  foreground?: string,
  auto_play_delay?: number
}

const props = defineProps<{
  conf: CarouselConf,
}>()


var intervalID

const fullScreenMode = ref(false)
const autoplay = ref(true)
const current_slide_num = ref(1)
const current_slide_url = ref()
const current_slide_ref = ref()
// https://vuejs.org/guide/essentials/template-refs.html#refs-inside-v-for
//const slides = ref([])

function change(slide_num) {
  //console.log('slide_num', slide_num)
  current_slide_num.value = slide_num
  //current_slide.value = props.slides[current_slide_num.value-1]
}


onMounted(async () => {
  //console.log('props', props.auto_play)
  if (props.conf.auto_play_delay) {
    if (intervalID !== undefined) {
      clearInterval(intervalID)
    }
    intervalID = setInterval(slideNext, props.conf.auto_play_delay)
  }

})


watch(fullScreenMode, async (newQuestion, oldQuestion) => {
  if (fullScreenMode.value == true) {
    openFullscreen();
  }
  else {
    closeFullscreen();
  }
})

watch(autoplay, async (newQuestion, oldQuestion) => {
  if (props.conf.auto_play_delay) {
    if (autoplay.value == true) {
      intervalID = setInterval(slideNext, props.conf.auto_play_delay)
    } else {
      if (intervalID !== undefined) {
        clearInterval(intervalID)
      }
    }
  }
})


function slideNext() {
  current_slide_num.value += 1
  if (current_slide_num.value > props.conf.slides.length) {
    current_slide_num.value = 1
  }
  //current_slide.value = props.conf.slides[current_slide_num.value-1]
  //history.pushState(null, null, `#item${current_slide.value}`)
  //location.hash = `#item${current_slide_num.value}`

  current_slide_url.value = props.conf.slides[current_slide_num.value - 1]
  //current_slide.value.task()
  //current_slide_ref.value.task()
  //console.log(`slideNext ${current_slide_num.value}`)
}

function openFullscreen() {
  var elem = document.documentElement;
  if (elem.requestFullscreen) {
    elem.requestFullscreen();
  } else if (elem.mozRequestFullScreen) {
    elem.mozRequestFullScreen();
  } else if (elem.webkitRequestFullscreen) {
    elem.webkitRequestFullscreen();
  } else if (elem.msRequestFullscreen) {
    elem.msRequestFullscreen();
  }

}

function closeFullscreen() {

  if (document.exitFullscreen) {
    document.exitFullscreen();
  } else if (document.mozCancelFullScreen) {
    document.mozCancelFullScreen();
  } else if (document.webkitExitFullscreen) {
    document.webkitExitFullscreen();
  } else if (document.msExitFullscreen) {
    document.msExitFullscreen();
  }

}

function reload() {
  // relad all slides

}

</script>


<template>
  <ClientOnly>
    <div>
      <div class="w-full">
        <ppw-anim-bullets v-if="props.conf?.background" :href="props.conf.background" class="w-full absolute" />
        <ppw-anim-bullets :href="current_slide_url" ref="current_slide_ref" class="w-full" />
        <ppw-anim-bullets v-if="props.conf?.foreground" :href="props.conf.foreground" class="w-full absolute" />

        <ppw-debug-panel>
          <p>anim bullet debug panel ....</p>
          <p> current_slide_num: {{ current_slide_num }}</p>
          <p> current_slide: {{ current_slide_url }}</p>
          <p> intervalID: {{ intervalID }}</p>
          <p> fullScreenMode: {{ fullScreenMode }}</p>
          <p> autoplay: {{ autoplay }}</p>


          <button @click="fullScreenMode = !fullScreenMode" class="btn">
            Fullscreen
          </button>
          <button @click="autoplay = !autoplay" class="btn">
            autoplay
          </button>
          <button @click="reload" class="btn">
            reload
          </button>
        </ppw-debug-panel>
      </div>
      <div class="flex justify-center w-full py-2 gap-2">
        <a v-for="(slide, slide_num) in props.conf.slides" :href="`#item${slide_num + 1}`"
          :class="(slide_num + 1) === current_slide_num ? 'btn-sm' : 'btn-xs'" class="btn"
          @click="change(slide_num + 1)">{{
            slide_num + 1 }}</a>
      </div>
    </div>
  </ClientOnly>
</template>
