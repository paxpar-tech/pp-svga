<script setup  lang="ts">
/*

cf https://greensock.com/docs/v3/GSAP/Timeline
cf https://greensock.com/docs/v3/GSAP/Timeline
cf https://github.com/slidevjs/slidev
cf https://codepen.io/tigt/post/improving-svg-rendering-performance
cf https://stackoverflow.com/questions/53862125/svg-why-does-getboundingclientrect-return-190-when-y-attribute-is-200

svg.querySelector('svg *|page')


svg.querySelector('@namespace inkscape "http://www.inkscape.org/namespaces/inkscape"; svg inkscape|page')


xmlns:inkscape="http://www.inkscape.org/namespaces/inkscape"
*/
import { ref, reactive, onMounted } from 'vue'
import gsap from "gsap"

export interface Transpose {
    color?: Array<Map<string, string>>;
    logo?: Array<Map<string, string>>;
    text?: Array<Map<string, string>>;
}

export interface TransposeSpot {
    color?: string;
    logo?: string;
    text?: string;
}

export interface AnimBulletProps {
    href: string;
    transposeDef?: Transpose;
    transposeSpot?: TransposeSpot;
}


const props = defineProps<AnimBulletProps>()

const MAX_STAGES = 20
const PAGE_DELAY = 1 // s
const STAGE_DELAY = 1 // s
const STAGGER_DELAY = 0.1 // s

const { $fetch2 } = useNuxtApp()

const tl = gsap.timeline({ repeat: -1, repeatDelay: 2 })


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
const speed = ref(1.0)

const applyTranspose = false
const toast = useToast()

watch(props, async (newValue, oldValue) => {
    //console.log({newValue, oldValue})
    await loadSVG()
})


watch(autoplay, async (newValue, oldValue) => {
    console.log("autoplay", { newValue, oldValue })
    if (newValue) {
        tl.play()
    } else {
        tl.pause()
    }
})


watch(speed, async (newValue, oldValue) => {
    //console.log("autoplay", { newValue, oldValue })
    tl.timeScale(newValue)
})



onMounted(async () => {
    await loadSVG()
})



const loadSVG = async () => {
    if (props.href) {
        //console.log(`loading ${props.href} ...`, props.href)

        try {
            // https://philippe.paxpar.tech/api/ref/public
            var data = await $fetch2(props.href, { parseResponse: txt => txt })
            raw_svg.value = data
            // we can't have valid ref now, let's delay a bit to fix this
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

//function svg_goto_page(page: PageInfo) {
function svg_goto_page(indexPage) {
    console.log(`goto page ${indexPage} ...`)
    if (autoplay.value) {
        tl.seek(`page${indexPage}`)
    } else {
        tl.seek(`page${indexPage + 1}`)
    }
    //tl.tweenTo(`page${indexPage}`)
    //tl.play(`page${indexPage}`)
}


const applyTransposeColor = (svg: SVGElement) => {
    const colorKeyTarget = props?.transposeSpot?.color
    // colorKeyTarget = 'cs'
    const colorsDef = props?.transposeDef?.color
    /*
    colorsDef = [{
        pp: "#655151ff",
        cs: "#655151ff",
    }, {
        pp: "#1197a563",
        cs: "#1197a563",
    }]
    */

    if ((colorKeyTarget === undefined) || (colorsDef === undefined)) {
        return
    }

    // loop over all color sets defined in transposeDef
    colorsDef.forEach((colorDef, index) => {
        /*
        colorDef = {
            pp: "#655151ff",
            cs: "#655151ff",
        }
        */
        // the target color for this key
        const colorTarget = colorDef[colorKeyTarget]
        const colorKeys = Object.keys(colorDef)
        // loop over all key in color set
        colorKeys.forEach((colorKey, index) => {
            // colorKey = 'pp'
            if (colorKey !== colorKeyTarget) {
                const colorSource = colorDef[colorKey]
                if (colorSource !== colorTarget) {
                    console.log(`transpose color ${colorKeyTarget} from ${colorSource} to ${colorTarget}`)
                    /*
                    fill:#1197a5;fill-opacity:0.38924;stroke:none;stroke-width:1.565;stroke-dasharray:none;stroke-opacity:1;stop-color:#000000
                    */
                    //const nodes = svg.querySelectorAll(`svg [ref='stage${stage}']`)
                    const nodes = svg.querySelectorAll(`svg [style*="fill:#1197a5"]`)
                    nodes.forEach((node: SVGElement, index) => {
                        const style = node.attributes['style']
                        console.log('color transpose on ', node, style)
                        node.style.fill = colorTarget
                    })
                }
            }
        })
    })
}

const applyTransposeLogo = (svg: SVGElement) => {
    if (props?.transposeSpot?.logo) {
        console.log(`apply transpose logo ${props.transposeSpot.logo}`)
    }
}

const applyTransposeText = (svg: SVGElement) => {
    if (props?.transposeSpot?.text) {
        console.log(`apply transpose text ${props.transposeSpot.text}`)
    }
}

const loadSVG2 = () => {
    const svg = container.value.querySelector("svg")
    //console.log({ svg })

    /*
    TODO: better way to fix the width/height values

    width="3840"     -->     width="100%"
    height="2160"    -->     height="auto"
    */
    //svg.setAttribute('width', '100%')
    svg.setAttribute('width', 'auto')
    svg.setAttribute('height', 'auto')
    console.log('svg', svg)


    const deck: DeckInfo = {
        node: svg,
        pages: [],
    }

    /*
    // fix href for images
    const nodes = svg.querySelectorAll(`svg image`)
    nodes.forEach((node, index) => {
        if (node.hasAttribute('xlink:href')) {
            const old_href = node.getAttribute('xlink:href')

        }
    })
    */


    if (applyTranspose) {
        applyTransposeColor(svg)
        applyTransposeLogo(svg)
        applyTransposeText(svg)
    }


    // get all pages
    const pages_nodes = svg.children[0].children
    // cf https://stackoverflow.com/questions/35969974/foreach-is-not-a-function-error-with-javascript-array
    //pages_nodes.forEach((pages_node, index) => {
    Array.prototype.forEach.call(pages_nodes, (pages_node, index) => {
        const rect = svg.createSVGRect()
        rect.x = pages_node.getAttribute('x')
        rect.y = pages_node.getAttribute('y')
        rect.width = pages_node.getAttribute('width')
        rect.height = pages_node.getAttribute('height')
        const page: PageInfo = {
            rect,
            viewBox: rect2viewbox(rect),
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
                //const inPage = interRect(node.getBBox(), page.rect)
                //const inPage = interRect(getRectPosition(node), page.rect)
                const inPage = rectCenterInRect(getRectPosition(node), page.rect)
                //const inPage = interRect(node.getBoundingClientRect(), page.rect)
                const nodeBBox = node.getBBox()
                const pageRect = page.rect
                //const pageBBox = page.getBBox()
                //console.log({ stage, inPage, nodeBBox, pageRect })
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

    // fill timeline
    // for each page
    deck.pages.forEach((page, indexPage) => {
        //tl.from(deck_info.value.node, {viewBox: page.viewBox})
        //tl.addLabel(`page${indexPage+1}`)
        tl.to(deck_info.value.node, {
            attr: { viewBox: page.viewBox },
            duration: PAGE_DELAY,
        }, `page${indexPage + 1}`)

        page.stages.forEach((stage, indexStage) => {
            tl.from(stage.nodes, {
                //y: 200,
                opacity: 0,
                duration: STAGE_DELAY,
                stagger: STAGGER_DELAY,
            })
        })
    })
    tl.restart()
    //console.log("deck_info", deck_info.value)
}

/*
 2 rect intersects if :
    global width < sum of width
    and
    global height < sum of height
I don't understand how checkIntersection() works so I did this :-/
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

/*
Check if the center of rect r1 is in rect r2
It is a better strategy than interRect() since an object (r1) can span on several pages (r2)
The object belongs to the page where its center is
*/
function rectCenterInRect(r1: SVGRect, r2: SVGRect) {
    const centerX = r1.x + r1.width / 2
    const centerY = r1.y + r1.height / 2
    return (r2.x <= centerX) && (centerX <= r2.x + r2.width) &&
        (r2.y <= centerY) && (centerY <= r2.y + r2.height)
}


/*

cf https://stackoverflow.com/questions/53635449/get-absolute-coordinates-of-element-inside-svg-using-js
cf https://www.w3.org/Graphics/SVG/IG/resources/svgprimer.html
*/
function rect2viewbox(rect) {
    return `${Math.round(rect.x)} ${Math.round(rect.y)} ${Math.round(rect.width)} ${Math.round(rect.height)}`
}

function getRectPosition(node) {
    //const rect = node.getBoundingClientRect()
    const rect = node.getBBox()
    //var elem = document.getElementById(circleElemId);
    var svg = node.ownerSVGElement;

    // Get the cx and cy coordinates
    var pt = svg.createSVGPoint()
    pt.x = rect.x;
    pt.y = rect.y;

    const pt1 = pt.matrixTransform(getTransformToElement(node, svg));

    pt.x = rect.x + rect.width;
    pt.y = rect.y + rect.height;

    const pt2 = pt.matrixTransform(getTransformToElement(node, svg));

    var rect2 = svg.createSVGRect()
    rect2.x = pt1.x
    rect2.y = pt1.y
    rect2.width = pt2.x - pt1.x
    rect2.height = pt2.y - pt1.y

    //return rect.matrixTransform(getTransformToElement(node, svg));
    return rect2
}


function getPagePosition(rect) {
    return rect
}

function getTransformToElement(fromElement, toElement) {
    return toElement.getCTM().inverse().multiply(fromElement.getCTM());
};


//var pos = getCirclePosition("thecircle");
//console.log("Coordinates are: " + pos.x + "," + pos.y);



</script>


<template>
    <div class="stack w-full">
        <h1>XXXXX</h1>
        <div ref="container" class="object-contain w-full" v-motion-fade v-html="raw_svg"></div>
    </div>
    <div class="flex my-4 w-full">

        <!-- play / pause -->
        <button @click="autoplay = !autoplay" class="btn px-6 mx-4">
            <icon :name="autoplay ? 'mdi:pause' : 'mdi:play'" size=32 />
        </button>

        <!-- go to page buttons -->
        <button v-for="(page, indexPage) in deck_info.pages" @click="svg_goto_page(indexPage + 1)"
            class="btn px-2 m-1">{{
    indexPage + 1
            }}</button>

        <!-- reload button -->
        <button @click="loadSVG" class="btn px-2 mx-4">
            <icon name="oi:reload" size=32 />
        </button>

        <!-- speed buttons -->
        <button @click="speed = 0.5" class="btn px-2 mx-1" :class="speed == 0.5 ? '' : 'btn-outline'">
            <icon name="fluent:animal-turtle-16-filled" size=32 />
        </button>
        <button @click="speed = 1" class="btn px-2 mx-1" :class="speed == 1 ? '' : 'btn-outline'">
            <icon name="fluent:animal-rabbit-24-filled" size=32 />
        </button>


    </div>


    <ppw-debug-panel>
        <p> transposeDef = {{ transposeDef }}</p>
        <p> transposeSpot = {{ transposeSpot }}</p>
        <button class="btn" @click="loadSVG2">loadSVG2</button>
        <h1>loaded SVG </h1>
        <ul class="pl-4">
            <template v-for="(page, index) in deck_info.pages">
                <li>page {{ index + 1 }}, viewBox {{ rect2viewbox(page.rect) }}</li>
                <ul class="pl-4">
                    <template v-for="(stage, index2) in page.stages">
                        <li>stage {{ index2 + 1 }}</li>
                        <ul class="pl-4">
                            <li v-for="(node, index3) in stage.nodes" :set="bbox = getRectPosition(node)">
                                node {{ index3 + 1 }},
                                id:{{ node?.id }},
                                bbox: {{ rect2viewbox(bbox) }},
                                <!--bbox {{ bbox.x }}, {{ bbox.y }} -->
                            </li>
                        </ul>
                    </template>
                </ul>
            </template>
        </ul>
    </ppw-debug-panel>
</template>

<style>

</style>
