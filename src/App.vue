<template>
    <div class="ball">
        {{ currentNum }}
    </div>
    <button @click="start">{{ drawing ? '停止' : '開始' }}</button>
    <div class="balls">
        <div
            :class="['ball', { spacial: index === maxBall - 1 }]"
            v-for="(ball, index) in showBalls"
            :key="ball">
            {{ ball }}
        </div>
    </div>
</template>

<script>
import { onMounted, ref, watch } from 'vue';

export default {
    name: 'App',
    setup () {
        const maxNum = 49; // 開獎號碼 1 ~ n
        const maxBall = 7; // 開獎球數
        const drawing = ref(false); // 是否開獎中
        const currentNum = ref(1); // 開獎動畫數字
        const currentBall = ref(0); // 第 n 個已開獎的球
        const selectedNum = []; // 全部的中獎號碼
        const showBalls = ref([]); // 已顯示的球

        let initTimer;
        const init = () => {
            initTimer = setInterval(() => {
                currentNum.value = currentNum.value < maxNum ? currentNum.value + 1 : 1;
            }, 500);
        };

        const start = () => {
            if (!drawing.value) {
                clearInterval(initTimer);
                drawing.value = true;
                draw();
            }
            else if (drawing.value && confirm('確定停止?')) {
                drawing.value = false;
                currentNum.value = 1;
                currentBall.value = 0;
                showBalls.value.length = 0;
                init();
            }
        };

        const draw = () => {
            // 產生所有數字
            const allNum = [];
            for (let n = 1; n <= maxNum; n++) {
                allNum.push(n);
            }

            // 產生所有數字中隨機選中的數字
            for (let i = 0; i < maxBall; i++) {
                const index = Math.floor(Math.random() * allNum.length);
                selectedNum.push(allNum.splice(index, 1)[0]);
            }

            generateBall();
        };

        const generateBall = () => {
            // 開獎中動畫
            const generateTimer1 = setInterval(() => {
                if (!drawing.value) {
                    clearInterval(generateTimer1);
                    return;
                }
                currentNum.value = currentNum.value < maxNum ? currentNum.value + 1 : 1;
            }, 50);

            // 兩秒後結束動畫並顯示球
            const generateTimer2 = setTimeout(() => {
                if (!drawing.value) {
                    clearInterval(generateTimer2);
                    return;
                }

                clearInterval(generateTimer1);
                showBalls.value.push(selectedNum[currentBall.value]);

                if (showBalls.value.length === maxBall) return;

                // 延遲一秒後顯示下一個
                const generateTimer3 = setTimeout(() => {
                    if (!drawing.value) {
                        clearInterval(generateTimer3);
                        return;
                    }
                    currentBall.value++;
                }, 1000);
            }, 2000);
        };

        watch(currentBall, value => {
            if (value) {
                generateBall();
            }
        });

        onMounted(() => {
            init();
        });

        return {
            currentNum,
            start,
            drawing,
            showBalls,
            maxBall
        };
    }
};
</script>

<style>
.ball {
    width: 80px;
    height: 80px;
    border-radius: 50%;
    background-color: orange;
    color: #fff;
    text-align: center;
    font-weight: bold;
    font-size: 40px;
    line-height: 80px;
}
.ball.spacial {
    background-color: red;
}
.balls {
    display: flex;
}
</style>
