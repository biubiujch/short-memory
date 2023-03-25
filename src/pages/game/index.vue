<template>
  <view class="content">
    <text class="level">level &nbsp;&nbsp;&nbsp; {{ level }}</text>
    <view v-if="time === 0" class="tips">
      <text class="text nextbtn" @tap="nextLevel" v-if="bingoIndex.length === 10">下一关</text>
      <text class="text nextbtn" v-else-if="errorIndex !== -1" @tap="tryAgain">再试一次</text>
      <text class="text" v-else>开始！！！</text>
    </view>
    <view v-else class="tips">
      <text class="text">tips:&nbsp;请记下所有数字的位置</text>
      <text class="text">&nbsp;{{ time }}&nbsp;秒后，按照1-10的顺序点击</text>
    </view>
    <view class="card-wrap">
      <view v-for="(number, index) of numbers" class="card" :key="index" @tap="() => handleClick(number, index)">
        <view class="inner flipBack" v-if="isStart && record[index] === 0">?</view>
        <view :class="innerStyle(number, index)" v-else>{{ number === -1 ? "" : number }}</view>
      </view>
    </view>
  </view>

  <uni-popup ref="popup" background-color="transparent">
    <view class="popup-content">
      <text class="text">GAME OVER!</text>
      <text class="retry" @tap="tryAgain">再试一次</text>
    </view>
  </uni-popup>
  <uni-popup ref="nextpop" background-color="transparent">
    <view class="popup-content">
      <text class="text">Congratulations！</text>
      <text class="retry" @tap="nextLevel">下一关</text>
    </view>
  </uni-popup>
</template>

<script lang="ts" setup>
import { onUnload } from "@dcloudio/uni-app";

import { computed, onMounted, ref, watch } from "vue";

function randomArray(n: number, m: number) {
  if (n > m) {
    throw new Error("n 不能大于 m");
  }

  // 生成一个包含1到n的数字数组
  const nums = Array.from({ length: n }, (_, i) => i + 1);

  // 随机打乱数字数组的顺序
  for (let i = 0; i < n - 1; i++) {
    const j = Math.floor(Math.random() * (n - i)) + i;
    [nums[i], nums[j]] = [nums[j], nums[i]];
  }

  // 将随机排列的数字填充到数组中
  const arr = Array.from({ length: m }, (_, i) => (i < n ? nums[i] : -1));

  // 随机打乱-1的位置
  for (let i = m - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [arr[i], arr[j]] = [arr[j], arr[i]];
  }

  return arr;
}
const timer = ref<NodeJS.Timeout | null>(null);
const time = ref<number>(20);
const isStart = ref<boolean>(false);
const round = ref<number>(1);
const numbers = ref<number[]>(randomArray(10, 16));
const record = ref<number[]>(Array.from({ length: 16 }, () => 0));
const errorIndex = ref<number>(-1);
const bingoIndex = ref<number[]>([]);
const popup = ref<any>(null);
const nextpop = ref<any>(null);
const level = ref<number>(1);

const innerStyle = (number: number, index: number) => {
  return index === errorIndex.value
    ? "inner flipFront error"
    : bingoIndex.value.findIndex((value) => value === index) !== -1
    ? "inner flipFront bingo"
    : "inner flipFront";
};

onMounted(() => {
  startGame();
});

const startGame = () => {
  timer.value = setInterval(() => {
    if (time.value == 0) {
      timer.value && clearInterval(timer.value);
      timer.value = null;
      return;
    }
    time.value -= 1;
  }, 1000);
};

watch(time, () => {
  if (time.value === 0) {
    isStart.value = true;
  }
});

const handleClick = (number: number, index: number) => {
  if (!isStart.value) {
    return;
  }
  if (bingoIndex.value.findIndex((value) => value === index) !== -1) {
    return;
  }
  record.value[index] = number;
  if (round.value !== number) {
    // gameover
    errorIndex.value = index;
    isStart.value = false;
    popup.value?.open();
    return;
  }
  if (number === 10) {
    // 过关
    isStart.value = false;
    nextpop?.value.open();
  }
  bingoIndex.value.push(index);
  round.value += 1;
};

// 通关
const nextLevel = () => {
  nextpop?.value.close();
  level.value += 1;
  handleReset();
  startGame();
};

// 重置
const handleReset = () => {
  numbers.value = randomArray(10, 16);
  isStart.value = false;
  round.value = 1;
  record.value = Array.from({ length: 16 }, () => 0);
  errorIndex.value = -1;
  bingoIndex.value = [];
  time.value = level.value < 2 ? 30 - level.value * 10 : 5;
};

const tryAgain = () => {
  popup.value?.close();
  handleReset();
  startGame();
};

onUnload(() => {
  timer.value && clearInterval(timer.value);
});
</script>

<style lang="scss" scoped>
.content {
  height: 100vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  justify-content: center;
  background-color: $uni-color-black;
  .level {
    color: $uni-color-green;
    margin-bottom: 20px;
    text-align: center;
  }
  .nextbtn {
    border-bottom: 1px solid $uni-color-green;
  }
  .tips {
    display: flex;
    align-items: center;
    flex-direction: column;
    .text {
      color: $uni-color-green;
      margin-bottom: 20px;
      text-align: center;
    }
  }

  .card-wrap {
    height: 100vw;
    display: grid;
    align-items: center;
    justify-items: center;
    grid-template-columns: repeat(4, 25%);
    grid-template-rows: repeat(4, 25%);

    .card {
      display: flex;
      align-items: center;
      justify-content: center;
      perspective: 1000px; /* 设置3D视距，使元素看起来更立体 */
      transition: transform 1s;
    }

    .inner {
      width: 16vw;
      height: 16vw;
      border-radius: 5px;
      box-sizing: border-box;
      backface-visibility: hidden;
      font-size: 2rem;
      display: flex;
      align-items: center;
      background-color: #fff;
      justify-content: center;
      transform: rotateY(0deg); /* 初始状态为正面朝上 */
    }
    .bingo {
      background-image: radial-gradient(circle at 50% 50%, #fff 40%, $uni-color-green);
    }
    .error {
      background-image: radial-gradient(circle at 50% 50%, #fff 40%, red);
    }
    .flipFront {
      animation: 0.5s flipFront;
    }
    .flipBack {
      animation: 0.5s flipBack;
    }

    @keyframes flipFront {
      from {
        transform: rotateY(180deg);
      }
      to {
        transform: rotateY(0deg);
      }
    }

    @keyframes flipBack {
      from {
        transform: rotateY(0deg);
      }
      to {
        transform: rotateY(180deg);
      }
    }
  }
}

.popup-content {
  background-color: rgba($color: #000000, $alpha: 0.3);
  height: 30vh;
  width: 100vw;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  color: $uni-color-green;
  .text {
    color: $uni-color-green;
    font-size: 2rem;
    margin-bottom: 20px;
    display: block;
  }
  .retry {
    display: block;
    font-size: 2.5rem;
    color: $uni-color-green;
    border-bottom: 2px solid $uni-color-green;
  }
}
</style>
