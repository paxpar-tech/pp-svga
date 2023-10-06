<script setup lang="ts">
//import 'https://flackr.github.io/scroll-timeline/dist/scroll-timeline.js';
useHead({
    script: [
        "https://flackr.github.io/scroll-timeline/dist/scroll-timeline.js",
    ]
})

/*
cf https://scroll-driven-animations.style/?debug=
cf https://www.youtube.com/watch?v=VgS5CP7zlXE&t=1s
cf https://www.youtube.com/watch?v=-pDPASqX97w
cf https://www.youtube.com/watch?v=iEPK5fppX8w
cf https://css-tricks.com/scale-svg/
*/

const svg = ref<SVGElement>()
const scroll = ref(0)
const scrollSections = ref<NodeListOf<HTMLElement>>(document.querySelectorAll('xxxx'))
const svgSections = ref<NodeListOf<HTMLElement>>(document.querySelectorAll('xxxx'))


const scrollSectionPositions = ref({})


onMounted(async () => {
})


function customAnimation() {
    svgSections.value.forEach((svgSection, sectionIndex) => {
        const scrollSection = scrollSections.value[sectionIndex]
        svgSection.style.opacity = "0.0"


        svgSection.animate({
            transform: [
                'translateX(300px)',
                'translateX(0px)',
            ],
            opacity: [0, 1],
        }, {
            timeline: new ViewTimeline({
                subject: scrollSection,
            }),
            //rangeStart: 'entry 25%',
            //rangeEnd: 'cover 50%',
            fill: "forwards",
        });

    })


}

function svgLoaded() {
    if (svg.value) {

        console.info('SVG loaded, starting scroll animation ...', svg.value)
        scrollSections.value = document.querySelectorAll('.scroll-section')
        svgSections.value = svg.value.querySelectorAll(`svg [inkscape\\:label^='stage']`)
        //console.log("svgSections", svgSections.value)

        customAnimation()

    }
}

</script>


<template>
    <div class="hero-content flex-col lg:flex-row">
        <div class="w-1/2">

            <!-- the SVG anim should not start here, so no scroll-section-->
            <div class="noscroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>

            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
            <div class="scroll-section">
                <h1>Lorem ipsum dolor sit amet</h1>
                <p>Nam sit amet vulputate magna. Curabitur molestie urna in auctor faucibus. Cras ut purus ac eros iaculis
                    dapibus
                    sit amet quis ante. Suspendisse potenti. Nulla mattis ut elit eget gravida. Nam fermentum et quam non
                    mollis.
                    Cras rhoncus consectetur arcu, in dictum est dictum ut. Nullam cursus lectus ac mi iaculis, eget
                    tincidunt
                    orci</p>
            </div>
        </div>
        <div class="w-1/2">
            <ppw-svg v-model="svg" @loaded="svgLoaded" class="background-anim" href="/scrollexample/anim.svg" />
        </div>

    </div>
</template>


<style>
/* default
:root {
    --scroll: 0;
}
*/


.scroll-section {
    height: 100vh;
    /* background-color: green; */
}


.noscroll-section {
    height: 100vh;
    /* background-color: green; */
}

.background-anim {
    /*
    height: 300px;
    width: 300px;
    */
    height: 100%;
    width: 50%;
    /*
    background-color: yellow;
    translate: 0% 10%;
    */
    position: fixed;
    top: 90px;
    /* transform: rotateX(60deg) rotateY(0deg) rotateZ(-45deg); */
    /* transform: rotateX(60deg) rotateY(0deg) rotateZ(-45deg); */
    transform: rotate(-20deg) skew(10deg) scaleY(1);

}
</style>