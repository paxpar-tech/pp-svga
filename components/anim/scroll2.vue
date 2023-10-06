<script setup lang="ts">
const props = withDefaults(
  defineProps<{
    href: string
    show_hidden_layers?: boolean
  }>(),
  {
    show_hidden_layers: false,
  }
)

//import 'https://flackr.github.io/scroll-timeline/dist/scroll-timeline.js';
useHead({
  script: ["https://flackr.github.io/scroll-timeline/dist/scroll-timeline.js"],
})

/*
cf https://scroll-driven-animations.style/?debug=
cf https://www.youtube.com/watch?v=VgS5CP7zlXE&t=1s
cf https://www.youtube.com/watch?v=-pDPASqX97w
cf https://www.youtube.com/watch?v=iEPK5fppX8w
cf https://css-tricks.com/scale-svg/
*/

// the SVG DOM node that will contain the animation
const svg = ref<SVGElement>()
// we start with empty NodeList :
const htmlSections = ref<NodeListOf<HTMLElement>>(document.querySelectorAll("xxxx"))
const svgSections = ref<NodeListOf<HTMLElement>>(document.querySelectorAll("xxxx"))
const svgHiddenSections = ref<NodeListOf<HTMLElement>>(document.querySelectorAll("xxxx"))

function customAnimation() {
  // hide 'hidden' svg layers
  if (props.show_hidden_layers === false) {
    svgHiddenSections.value.forEach((section, sectionIndex) => {
      // the SVG layer should be hidden
      section.style.opacity = "0.0"
    })
  }

  // animate svg layers
  // they appear when DOM section scroll
  // loop over the SVG sections
  svgSections.value.forEach((svgSection, sectionIndex) => {
    // the corresponding n-th DOM section
    const scrollSection = htmlSections.value[sectionIndex]
    // the SVG layer should start hidden
    svgSection.style.opacity = "0.0"

    svgSection.animate(
      {
        transform: ["translateX(300px)", "translateX(0px)"],
        opacity: [0, 1],
      },
      {
        timeline: new ViewTimeline({
          subject: scrollSection,
        }),
        rangeStart: "entry 0%",
        rangeEnd: "cover 20%",
        fill: "forwards",
      }
    )
  })

  // animation DOM sub sections
  // they appear when their DOM section scoll
  // loop over the DOM sections
  htmlSections.value.forEach((scrollSection, sectionIndex) => {
    // the corresponding DOM sub-sections
    const scrollSubSections = scrollSection.querySelectorAll(".scroll-sub-section")
    //console.log(`sub-sections of section ${sectionIndex}`, scrollSubSections);
    scrollSubSections.forEach((scrollSubSection, subSectionIndex) => {
      const rangeStartValue = 20 + subSectionIndex * 10
      const rangeEndValue = 50 + subSectionIndex * 10
      console.log("range", rangeStartValue, rangeEndValue)
      scrollSubSection.animate(
        {
          transform: ["translateX(200px)", "translateX(0px)"],
          opacity: [0, 1],
        },
        {
          timeline: new ViewTimeline({
            subject: scrollSection,
          }),
          // rangeStart: "entry 0%",
          rangeStart: `entry ${rangeStartValue}%`,
          // rangeEnd: "cover 80%",
          rangeEnd: `cover ${rangeEndValue}%`,
          fill: "forwards",
        }
      )
    })
  })
}

// Called when the SVG is fully loaded
function svgLoaded() {
  if (svg.value) {
    console.info("SVG loaded, starting scroll animation ...", svg.value)
    htmlSections.value = document.querySelectorAll(".scroll-section")
    svgSections.value = svg.value.querySelectorAll("svg [inkscape\\:label^='stage']")
    svgHiddenSections.value = svg.value.querySelectorAll(
      "svg [inkscape\\:label^='hidden']"
    )

    //console.log("svgSections", svgSections.value)

    customAnimation()
  }
}
</script>

<template>
  <div class="hero-content flex-col lg:flex-row">
    <div class="w-1/2">
      <slot />
    </div>
    <div class="w-1/2">
      <button class="btn btn-circle btn-outline">
        <icon inline name="mdi:shield-bug" size="24"></icon>
      </button>
      <ppw-svg v-model="svg" @loaded="svgLoaded" class="background-anim" :href="href" />
    </div>
  </div>
</template>

<style>
.background-anim {
  /*
    height: 300px;
    width: 300px;
    height: 100%;
    width: 50%;
    */
  /* background-color: rgba(255, 255, 0, 0.353);*/
  /*
    translate: 0% 10%;
    */
  position: fixed;
  /* top: 90px; */
  top: 50%;

  /* transform: rotateX(60deg) rotateY(0deg) rotateZ(-45deg); */
  /* transform: rotateX(60deg) rotateY(0deg) rotateZ(-45deg); */
  /*
  transform: translate(0, 10vh) rotate(-20deg) skew(10deg) scaleY(1);
  transform: translate(0%, -50%) rotate(0deg) skew(0deg);
  */
  transition: all 1s;
  opacity: 1;
}
/*
.background-anim:visible {
  opacity: 1;
  transform: translate(0, 0) rotate(-20deg) skew(10deg) scaleY(1);
}
*/
</style>
