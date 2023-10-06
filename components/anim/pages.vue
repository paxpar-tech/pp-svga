<script setup  lang="ts">
/*

cf https://github.com/slidevjs/slidev

svg.querySelector('svg *|page')


svg.querySelector('@namespace inkscape "http://www.inkscape.org/namespaces/inkscape"; svg inkscape|page')


xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
*/
import { ref, reactive, onMounted } from 'vue'
import { useMotion } from '@vueuse/motion'
import { useIntervalFn } from '@vueuse/core'

export interface AnimBulletProps {
    href: string;
}

const MAX_STAGES = 20
const PAGE_DELAY = 1000 // ms
const STAGE_DELAY = 1000 // ms
const props = defineProps<AnimBulletProps>()
const { $fetch2 } = useNuxtApp()


interface StageInfo {
    nodes: HTMLElement[];
}

interface PageInfo {
    rect: SVGRect;
    viewBox: string;
    stages: StageInfo[];
}

interface DeckInfo {
    node: HTMLElement;
    pages: PageInfo[];
}


const raw_svg = ref()
const container = ref()
const svg_node = ref()
const deck_info = ref<DeckInfo>({
    node: undefined,
    pages: [],
})
const autoplay = ref(true)
const toast = useToast()

watch(props, async (newValue, oldValue) => {
    //console.log({newValue, oldValue})
    await loadSVG()
})


watch(autoplay, async (newValue, oldValue) => {
    console.log("autoplay", { newValue, oldValue })
    if (newValue) {
        //TODO: start timer
    } else {
        //TODO: kill timer
    }
})


onMounted(async () => {
    await loadSVG()
})



//const { pause, resume, isActive } = useIntervalFn(goto_next_page, 1000)


function goto_next_page() {
    console.log('goto next page ...')
    //#TODO
}

const loadSVG = async () => {
    if (props.href) {
        //console.log(`loading ${props.href} ...`, props.href)

        try {
            // https://philippe.paxpar.tech/api/ref/public
            var data = await $fetch2(props.href, { parseResponse: txt => txt })
            //var data = await $fetch2(svg_url, { parseResponse: txt => txt })
            //console.log({ data })
            //TODO: better do this in DOM rather than in raw string
            //data = data.replace('width="3840"', 'width="100%"')
            //data = data.replace('height="2160"', 'height="100%"')
            raw_svg.value = data
            /*
                TODO: better way to fix the width/height values

                width="3840"     -->     width="100%"
                height="2160"    -->     height="auto"
                */
            // we can have valid ref now
            setTimeout(loadSVG2)


        } catch (err) {
            toast.error(`Erreur lors de la récupération de l'animation ${props.href}`)
        }
    }
}


/*
get svg pages
used to set viewbpx viewBox="0 0 113.08411 75.306931"
*/
function svg_pages(svg) {
    console.log('dbg aaa', svg)
    const pages = []
    if (svg === undefined) {
        return pages
    }

    const pages_nodes = svg.children[0].children
    pages_nodes.forEach((pages_node, index) => {
        pages.push({
            x: pages_node.getAttribute('x'),
            y: pages_node.getAttribute('y'),
            width: pages_node.getAttribute('width'),
            height: pages_node.getAttribute('height'),
        })
    })


    console.log(pages)
    return pages
}


function svg_goto_page(page: PageInfo) {
    //console.log('goto page', page)
    //console.log('svg viewbox', svg.getAttribute('viewBox'))
    svg_start_anim(page)

    const node = deck_info.value.node
    const { variant } = useMotion(node, {
        initial: {
            opacity: 0,
            x: 500,
        },
        enter: {
            opacity: 1,
            x: 0,
            transition: {
                duration: PAGE_DELAY,
            },
        },
        //hovered: {
        //    scale: 1.2,
        //},
    })

    // move viewBox to the page
    deck_info.value.node.setAttribute('viewBox', page.viewBox)
}


function svg_start_anim(page: PageInfo) {
    page.stages.forEach((stage, index1) => {
        stage.nodes.forEach((node, index2) => {
            //const motionInstance = useMotion(node, {
            const { variant } = useMotion(node, {
                initial: {
                    opacity: 0,
                    y: 100
                },
                enter: {
                    opacity: 1,
                    y: 0,
                    transition: {
                        delay: PAGE_DELAY + index1 * STAGE_DELAY,
                    },
                },
                //hovered: {
                //    scale: 1.2,
                //},
            })
        })

    })
}

const loadSVG2 = () => {
    const svg = container.value.querySelector("svg")
    const deck: DeckInfo = {
        node: svg,
        pages: [],
    }

    // get all pages
    const pages_nodes = svg.children[0].children
    pages_nodes.forEach((pages_node, index) => {
        const rect = svg.createSVGRect()
        rect.x = pages_node.getAttribute('x')
        rect.y = pages_node.getAttribute('y')
        rect.width = pages_node.getAttribute('width')
        rect.height = pages_node.getAttribute('height')
        const page: PageInfo = {
            rect,
            viewBox: `${rect.x} ${rect.y} ${rect.width} ${rect.height}`,
            stages: [],
        }
        deck.pages.push(page)
    })


    // get stages per page
    for (var stage = 1; stage <= MAX_STAGES; stage++) {
        const nodes = svg.querySelectorAll(`svg [ref='stage${stage}']`)
        //console.log(`nodes stage ${stage}`, nodes)
        nodes.forEach((node, index) => {
            //console.log("loop over node", { stage, bounding, node })
            //node.checkEnclosure(node, )
            deck.pages.forEach((page, index) => {
                //const inPage2 = svg.checkIntersection(node, page.rect)
                const inPage = interRect(node.getBBox(), page.rect)
                const nodeBBox = node.getBBox()
                const pageRect = page.rect
                //const pageBBox = page.getBBox()
                console.log({ stage, inPage, nodeBBox, pageRect })
                //console.log(node.checkIntersection(page.rect))
                //console.log(node.checkVisibility())
                //console.log(node.getBBox())
                if (inPage) {
                    // fill empty stages
                    while (page.stages.length < stage) {
                        // add empty stage
                        page.stages.push({ nodes: [] })
                    }
                    page.stages[stage - 1].nodes.push(node)
                    //page.stages
                }
            })
        })
    }

    deck_info.value = deck
    svg_node.value = svg

    svg.setAttribute('width', '100%')
    svg.setAttribute('height', '100%')
    console.log("deck_info", deck_info.value)

}

/*
 2 rect intersects if :
    global width < sum of width
    and
    global height < sum of height
I don't understand how checkIntersection() works
*/
function interRect(r1: SVGRect, r2: SVGRect) {
    const sumW = r1.width + r2.width
    const sumH = r1.height + r2.height
    const globalW = Math.max(
        Math.abs((r2.x + r2.width) - r1.x),
        Math.abs((r1.x + r1.width) - r2.x),
    )
    const globalH = Math.max(
        Math.abs((r2.y + r2.height) - r1.y),
        Math.abs((r1.y + r1.height) - r2.y),
    )

    return ((globalW < sumW) && (globalH < sumH))
}


//defineExpose({
//    task,
//})

</script>


<template>
    <div class="stack w-full">
        <div ref="container" class="object-contain w-full" v-motion-fade v-html="raw_svg"></div>
    </div>
    <div>
        <div class="flex">
            <button @click="autoplay = !autoplay" class="btn btn-circle" :class="autoplay ? 'btn-outline' : ''">
                <icon name="pajamas:autoplay" size=32></icon>
            </button>

            <p>page</p>
            <button v-for="(page, index) in deck_info.pages" @click="svg_goto_page(page)" class="btn px-2 m-1">{{
            index + 1 }}</button>
        </div>
    </div>
    <ppw-debug-panel>
        <h1>SVG load done test </h1>
        <button class="btn" @click="loadSVG2">loadSVG2</button>
        <h1>loaded SVG </h1>
        <ul class="pl-4">
            <template v-for="(page, index) in deck_info.pages">
                <li>page {{ index + 1 }}</li>
                <ul class="pl-4">
                    <template v-for="(stage, index2) in page.stages">
                        <li>stage {{ index2 + 1 }}</li>
                        <ul class="pl-4">
                            <li v-for="(node, index3) in stage.nodes">node {{ index3 + 1 }}</li>
                        </ul>
                    </template>
                </ul>
            </template>
        </ul>
    </ppw-debug-panel>
</template>
