<template>
    <div class="ball">
        {{ currentNum }}
    </div>
    <button @click="start">{{ drawing ? '停止' : '開始' }}</button>
    <div class="list">
        <div
            :class="['ball', { spacial: index === maxBall - 1 }]"
            v-for="(ball, index) in ballList"
            :key="ball">
            {{ ball }}
        </div>
    </div>
    <button v-if="ballList.length" @click="toggleSortRule">依照開獎順序 / 依照編號排序</button>
</template>

<script>
import { computed, onMounted, ref } from 'vue';

export default {
    name: 'App',
    setup () {
        const maxNum = 49; // 開獎號碼 1 ~ n
        const maxBall = 7; // 開獎總球數
        const drawing = ref(false); // 是否正在開獎
        const currentBall = ref(0); // 目前開獎到第幾顆球
        const selectedNum = []; // 全部的開獎號碼
        const showBalls = ref([]); // 已顯示的球

        let initTimer;
        const init = () => {
            initTimer = setInterval(setCurrentNum, 500);
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
                selectedNum.length = 0;
                showBalls.value.length = 0;
                init();
            }
        };

        const draw = () => {
            // 產生所有數字
            const numbers = [];
            for (let n = 1; n <= maxNum; n++) {
                numbers.push(n);
            }

            // 產生所有數字中隨機選中的數字
            for (let i = 0; i < maxBall; i++) {
                const index = Math.floor(Math.random() * numbers.length);
                selectedNum.push(numbers.splice(index, 1)[0]);
            }

            generateBall();
        };

        // 開獎中動畫
        const generateBall = () => {
            let isShowBall = false;
            let delay = 10;
            const generating = () => {
                if (!drawing.value || isShowBall) return;

                setCurrentNum();
                setTimeout(generating, delay *= 1.1);

                // 停止動畫並顯示球
                if (delay > 500) {
                    isShowBall = true;
                    showBall();
                }
            };
            generating();
        };

        const showBall = () => {
            showBalls.value.push(selectedNum[currentBall.value]);

            if (showBalls.value.length === maxBall) return;

            // 顯示後延遲一秒再產生下一個
            setTimeout(() => {
                if (!drawing.value) return;
                currentBall.value++;
                generateBall();
            }, 1000);
        };

        // 開獎動畫數字
        const currentNum = ref(1);
        const setCurrentNum = () => {
            currentNum.value = currentNum.value < maxNum ? currentNum.value + 1 : 1;
        };

        // 排序
        const sortRule = ref(0); // 0:開獎順序 1:編號排序 (特別號固定最後)
        const ballList = computed(() => {
            if (sortRule.value === 0) {
                return showBalls.value;
            }
            else {
                const _showBalls = showBalls.value.slice(0, maxBall - 1);
                _showBalls.sort((a, b) => a - b);
                _showBalls.push(showBalls.value[maxBall - 1]);
                return _showBalls;
            }
        });
        const toggleSortRule = () => {
            sortRule.value = sortRule.value === 0 ? 1 : 0;
        };

        onMounted(() => {
            init();
        });

        return {
            currentNum,
            start,
            drawing,
            showBalls,
            maxBall,
            ballList,
            toggleSortRule
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
.list {
    display: flex;
}
</style>
