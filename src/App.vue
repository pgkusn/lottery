<template>
    <div class="ball">
        {{ currentNum }}
    </div>
    <button @click="start">{{ drawing ? '重新開始' : '開始' }}</button>
    <div class="list">
        <div :class="['ball', { spacial: ball.index === maxBall }]" v-for="ball in ballList" :key="ball.num">
            {{ ball.num }}
        </div>
    </div>
    <button v-if="ballList.length === maxBall" @click="toggleSortRule">依照開獎順序 / 依照編號排序</button>
</template>

<script>
import { computed, onMounted, ref } from 'vue';

export default {
    name: 'App',
    setup () {
        const maxNum = 49; // 開獎號碼 1 ~ n
        const maxBall = 7; // 總球數
        const drawing = ref(false); // 是否正在開獎
        const numbers = ref([]);
        const selectedNum = computed(() => numbers.value.filter(item => item.selected));
        const unselectedNum = computed(() => numbers.value.filter(item => !item.selected));

        let timer;
        const setTimer = () => {
            timer = setInterval(setCurrentNum, 500);
        };

        const start = () => {
            if (!drawing.value) {
                clearInterval(timer);
                drawing.value = true;
                draw();
            }
            else if (drawing.value && confirm('確定重新開始?')) {
                drawing.value = false;
                currentNum.value = 1;
                numbers.value.forEach(item => (item.selected = false));
                setTimer();
            }
        };

        const draw = () => {
            // 開獎中動畫
            let isShowBall = false;
            let delay = 10;
            const generateBall = () => {
                if (!drawing.value || isShowBall) return;

                setCurrentNum();
                setTimeout(generateBall, delay *= 1.1);

                // 停止動畫並顯示球
                if (delay > 500) {
                    isShowBall = true;
                    showBall();
                }
            };
            generateBall();
        };

        const showBall = () => {
            // 抽號碼
            unselectedNum.value.forEach(i => {
                if (i.num === currentNum.value) {
                    numbers.value.forEach(j => {
                        if (j.num === i.num) {
                            j.selected = true;
                            j.index = selectedNum.value.length;
                        }
                    });
                }
            });

            if (selectedNum.value.length === maxBall) return;

            // 延遲一秒再抽下一個
            setTimeout(() => {
                if (!drawing.value) return;
                draw();
            }, 1000);
        };

        // 開獎動畫數字
        const currentNum = ref(1);
        const setCurrentNum = () => {
            currentNum.value = currentNum.value < maxNum ? currentNum.value + 1 : 1;
        };

        // 排序
        const sortRule = ref(0); // 0:開獎順序(預設) 1:編號排序(特別號固定最後)
        const ballList = computed(() => {
            const _selectedNum = JSON.parse(JSON.stringify(selectedNum.value));
            if (sortRule.value === 0) {
                return _selectedNum.sort((a, b) => a.index - b.index);
            }
            else {
                const newSelectNum = _selectedNum.slice(0, maxBall - 1);
                newSelectNum.sort((a, b) => a.num - b.num);
                newSelectNum.push(_selectedNum[maxBall - 1]);
                return newSelectNum;
            }
        });
        const toggleSortRule = () => {
            sortRule.value = sortRule.value === 0 ? 1 : 0;
        };

        onMounted(() => {
            // 產生所有數字
            for (let n = 1; n <= maxNum; n++) {
                numbers.value.push({
                    num: n,
                    selected: false
                });
            }
            setTimer();
        });

        return {
            currentNum,
            start,
            drawing,
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
