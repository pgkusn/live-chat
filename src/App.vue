<template>
    <div class="relative flex flex-col h-full max-w-[414px] mx-auto overflow-hidden">
        <!-- time -->
        <div class="flex absolute top-8 left-1/2 z-10 -translate-x-1/2 text-white font-mono">
            <div ref="liveText" class="px-2 py-1 bg-red-300">
                LIVE
            </div>
            <div class="px-2 py-1 bg-black bg-opacity-80">
                {{ showTime }}
            </div>
        </div>

        <!-- video -->
        <div class="relative h-[450px] flex justify-center flex-shrink-0">
            <video ref="vid" class="h-full max-w-none" />
        </div>

        <!-- message -->
        <ul class="relative pt-4 px-4 pb-16 flex-grow overflow-auto">
            <li v-for="msg in messages" :key="msg.id" class="flex items-center mb-4 last:mb-0">
                <img :src="msg.picture" class="flex-shrink-0 rounded-full mr-4">
                <div>
                    <p class="font-bold">
                        {{ msg.name }}
                    </p>
                    <p>{{ msg.content }}</p>
                </div>
            </li>
        </ul>

        <!-- emoji clicked -->
        <ul>
            <transition-group @enter="emojiEnterHandler">
                <li v-for="item in emojiClicked" :key="item" class="absolute text-5xl top-[480px] right-16">
                    {{ item }}
                </li>
            </transition-group>
        </ul>

        <!-- emoji tool -->
        <ul class="absolute inset-x-0 bottom-0 p-2 flex justify-end text-4xl bg-white bg-opacity-90" @click="emojiClickHandler">
            <li class="p-1 cursor-pointer transition duration-75 active:scale-125 active:duration-[0s]">
                😀
            </li>
            <li class="p-1 cursor-pointer transition duration-75 active:scale-125 active:duration-[0s]">
                😅
            </li>
            <li class="p-1 cursor-pointer transition duration-75 active:scale-125 active:duration-[0s]">
                😂
            </li>
            <li class="p-1 cursor-pointer transition duration-75 active:scale-125 active:duration-[0s]">
                🥳
            </li>
            <li class="p-1 cursor-pointer transition duration-75 active:scale-125 active:duration-[0s]">
                👍
            </li>
        </ul>
    </div>
</template>

<script>
import { computed, onMounted, ref } from 'vue';
import { gsap } from 'gsap';
import messages from '@/assets/messages.json';

export default {
    setup () {
        const liveText = ref(null);
        const vid = ref(null);

        // time
        const currentTime = ref(0);
        const showTime = computed(() => {
            const sec = currentTime.value % 60;
            const min = Math.floor(currentTime.value / 60) % 60;
            const hour = Math.floor(currentTime.value / 3600) % 24;
            const pad = n => String(n).padStart(2, '0');
            return `${pad(hour)}:${pad(min)}:${pad(sec)}`;
        });
        setInterval(() => {
            currentTime.value++;
        }, 1000);

        // video
        navigator.mediaDevices
            .getUserMedia({ video: true })
            .then(function (mediaStream) {
                vid.value.srcObject = mediaStream;
                vid.value.onloadedmetadata = function (e) {
                    vid.value.play();
                };
            })
            .catch(function (err) {
                console.log(err.name + ': ' + err.message);
            });

        // emoji
        const emojiClicked = ref([]);
        const emojiClickHandler = () => {
            if (event.target.tagName === 'LI') {
                emojiClicked.value.push(event.target.textContent);
            }
        };
        const emojiEnterHandler = el => {
            const tl = gsap.timeline();
            tl.set(el, {
                scale: 0.2,
                x: Math.random() * -100 + 20
            }).to(el, {
                duration: 1,
                scale: 1,
                y: -100 + Math.random() * -100,
                ease: 'Power4.out'
            }).to(el, {
                duration: 1,
                scale: 0.6,
                x: -500,
                ease: 'Power1.in'
            });
        };

        onMounted(() => {
            gsap.to(liveText.value, {
                duration: 1,
                backgroundColor: '#f00',
                repeat: -1,
                ease: 'none',
                yoyo: true
            });
        });

        return {
            liveText,
            vid,
            showTime,
            emojiClicked,
            emojiClickHandler,
            emojiEnterHandler,
            messages
        };
    }
};
</script>

<style>
html,
body,
#app {
    height: 100%;
}
body {
    min-width: 37px;
}
</style>