<template>
  <view class="content">
    <view class="block">
      <view class="text" v-for="value of numbers" :key="value">
        {{ value }}
      </view>
    </view>
    <button class="start-btn" @tap="handleJump">开始</button>
  </view>
</template>

<script setup lang="ts">
import { onHide, onShow, onUnload } from "@dcloudio/uni-app";
import { onMounted, ref } from "vue";

const random = () => {
  let res: string[] = [];
  for (let index = 0; index < 30; index++) {
    res.push((Math.random() * 100000).toString(2));
  }
  return res;
};

const numbers = ref<string[]>(random());
const timer = ref<NodeJS.Timeout | null>(null);

onShow(() => {
  timer.value = setInterval(() => {
    numbers.value = random();
  }, 300);
});

onHide(() => {
  timer.value && clearInterval(timer.value);
  timer.value = null;
});

onUnload(() => {
  timer.value && clearInterval(timer.value);
  timer.value = null;
});

const handleJump = () => {
  uni.redirectTo({
    url: "/pages/game/index"
  });
};
</script>

<style scoped lang="scss">
.content {
  height: 100vh;
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-direction: column;
  z-index: 1;
  .block {
    position: absolute;
    z-index: -1;
    height: 100vh;
    width: 100vw;
    overflow: hidden;
    background-color: $uni-color-black;
    filter: blur(1px);
    .text {
      color: $uni-color-green;
    }
  }

  .start-btn {
    background-color: $uni-color-black;
    color: #fff;
    box-shadow: 0px 0px 3px $uni-color-green;
    border-radius: 10px;
    width: 100px;
    animation: breathing 2s ease-in-out infinite;
  }
  @keyframes breathing {
    0% {
      box-shadow: 0 0 3px 3px #ccc;
    }
    100% {
      box-shadow: 0 0 6px 3px $uni-color-green;
    }
  }
}
</style>
