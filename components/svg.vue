<script setup lang="ts">
const toast = useToast()

export interface AnimBulletProps {
  href: string
}

const props = defineProps<AnimBulletProps>()

const emit = defineEmits(["loaded", "update:modelValue"])

const raw_svg = ref()
const container = ref()

onMounted(async () => {
  if (props.href) {
    //console.log(`loading ${props.href} ...`, props.href)

    try {
      // https://philippe.paxpar.tech/api/ref/public
      var data = await $fetch(props.href, { parseResponse: (txt) => txt })
      //console.log({ data })
      data = data.replace('width="3840"', 'width="100%"')
      data = data.replace('height="2160"', 'height="100%"')
      raw_svg.value = data
      //  TODO: better way to fix the width/height values
      //
      //  width="3840"     -->     width="100%"
      //  height="2160"    -->     height="auto"
      //
      // we can have valid ref now
      setTimeout(task)
    } catch (err) {
      toast.error(`Erreur lors de la récupération de l'animation ${props.href}`)
    }
  }
})

const task = () => {
  const svg = container.value.querySelector("svg")
  // console.log({ svg })
  // svg.setAttribute('width', '100%')
  // svg.setAttribute('height', '100%')
  console.log({ svg })

  if (svg) {
    // emit and th svg node
    console.log("emit1 ...")
    emit("update:modelValue", svg)
    console.log("emit2 ...")
    emit("loaded", svg)
  }
}
</script>

<template>
  <div class="outer-div">
    <div ref="container" v-html="raw_svg" class="inner-div" v-motion-fade></div>
  </div>
</template>

<style scoped>
.outer-div {
  /* background-color: aquamarine; */
}

.inner-div {
  /* background-color: rgb(41, 57, 10); */
}
</style>
