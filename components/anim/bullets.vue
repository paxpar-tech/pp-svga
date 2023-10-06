<script setup  lang="ts">

import { ref, reactive, onMounted } from 'vue'
const { $fetch2 } = useNuxtApp()
import { useMotion } from "@vueuse/motion"


export interface AnimBulletProps {
    href: string;
}

const props = defineProps<AnimBulletProps>()
const toast = useToast()

//const svg_url = '/blueman.svg'
//const svg_url = '/anim_b1.svg'
const raw_svg = ref()
const container = ref()


watch(props, async (newValue, oldValue) => {
    //console.log({newValue, oldValue})
    await loadSVG()
})


onMounted(async () => {
    await loadSVG()
})

const loadSVG = async () => {
    if (props.href) {
        //console.log(`loading ${props.href} ...`, props.href)

        try {
            // https://philippe.paxpar.tech/api/ref/public
            var data = await $fetch(props.href, { parseResponse: txt => txt })
            //var data = await $fetch2(svg_url, { parseResponse: txt => txt })
            //console.log({ data })
            //TODO: better do this in DOM rather than in raw string
            data = data.replace('width="3840"', 'width="100%"')
            data = data.replace('height="2160"', 'height="100%"')
            raw_svg.value = data
            /*
                TODO: better way to fix the width/height values

                width="3840"     -->     width="100%"
                height="2160"    -->     height="auto"
                */
            // we can have valid ref now
            setTimeout(task)


        } catch (err) {
            toast.error(`Erreur lors de la récupération de l'animation ${props.href}`)
        }
    }
}

const task = () => {
    const svg = container.value.querySelector("svg")
    console.log("anim bullet task", { container, svg })
    // svg.setAttribute('width', '100%')
    // svg.setAttribute('height', '100%')
    // console.log({ svg })
    if (svg) {
        const items = []
        for (var i = 1; i < 20; i++) {
            const node = svg.querySelector(`svg [ref='item${i}']`)
            if (node) {
                items.push(node)
            }
            // document.querySelector("svg [ref='item2']"),
        }

        items.forEach((ref, index) => {
            const { variant } = useMotion(ref, {
                initial: {
                    y: 100,
                    opacity: 0,
                },
                enter: {
                    y: 0,
                    opacity: 1,
                    transition: {
                        type: "spring",
                        stiffness: 350,
                        damping: 20,
                        delay: 500 + index * 500,
                        onComplete: () => {
                            variant.value = "levitate";
                        },
                    },
                },
                levitate: {
                    y: 15,
                    transition: {
                        duration: 2000,
                        repeat: Infinity,
                        ease: "easeInOut",
                        repeatType: "mirror",
                    },
                },
            });
        })

    }

    if (svg) {
        const targets = []
        for (var i = 1; i < 20; i++) {
            const node = svg.querySelector(`svg [ref='target${i}']`)
            if (node) {
                targets.push(node)
            }
            // document.querySelector("svg [ref='item2']"),
        }


        targets.forEach((ref, index) => {
            const { variant } = useMotion(ref, {
                initial: {
                    y: 100,
                    opacity: 0,
                },
                enter: {
                    y: 0,
                    opacity: 1,
                    transition: {
                        type: "spring",
                        stiffness: 350,
                        damping: 20,
                        delay: 500 + index * 500,
                        onComplete: () => {
                            variant.value = "levitate";
                        },
                    },
                },
                levitate: {
                    y: 15,
                    transition: {
                        duration: 2000,
                        repeat: Infinity,
                        ease: "easeInOut",
                        repeatType: "mirror",
                    },
                },
            });
        });
    }

}

defineExpose({
    task,
})

</script>


<template>
    <div class="stack w-full">
        <ppw-debug-panel>
            <h1>SVG load done test </h1>
            <button class="btn" @click="task">XXXX</button>
            <h1>loaded SVG </h1>
        </ppw-debug-panel>

        <div ref="container" class="object-contain w-full" v-motion-fade v-html="raw_svg"></div>

    </div>
</template>
