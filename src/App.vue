
<template>
  <div class="read-clock">
    <clock
      :isDark="isDark"
      :size="clockSize"
      v-model:hours="hours"
      v-model:minutes="minutes"
      v-model:seconds="seconds"
      :showNumbers="showNumbers"
    />
    <div class="switchers">
      <switcher v-model="isDark">深色模式</switcher>
      <switcher v-model="showSeconds">显示秒针</switcher>
      <switcher v-model="showNumbers">显示刻度</switcher>
    </div>
    <div class="time">{{ time }}</div>
    <div class="button" @click="generate">生成题目</div>
    <div class="button" @click="showTime = true">查看答案</div>
  </div>
</template>

<script setup lang="ts">
import { computed, onMounted, ref, watch } from 'vue';
import Clock from './components/Clock.vue';
import Switcher from './components/Switcher.vue';

const clockSize = Math.min(
  500,
  window.innerWidth - 40,
  window.innerHeight - 40,
);
const hours = ref(1);
const minutes = ref(0);
const seconds = ref<number | undefined>(0);
const showTime = ref(true);
const showSeconds = ref(
  window.localStorage.getItem('readClock:showSeconds') === '1',
);
const showNumbers = ref(
  !(window.localStorage.getItem('readClock:showNumbers') === '0'),
);
const isDark = ref(window.localStorage.getItem('readClock:isDark') === '1');

onMounted(() => {
  generate();
});

watch(
  () => showSeconds.value,
  () => {
    seconds.value = showSeconds.value ? 0 : undefined;
    window.localStorage.setItem(
      'readClock:showSeconds',
      showSeconds.value ? '1' : '0',
    );
  },
  {
    immediate: true,
  },
);

watch(
  () => showNumbers.value,
  () => {
    window.localStorage.setItem(
      'readClock:showNumbers',
      showNumbers.value ? '1' : '0',
    );
  },
  {
    immediate: true,
  },
);

watch(
  () => isDark.value,
  () => {
    document.body.classList.toggle('dark', isDark.value);
    window.localStorage.setItem('readClock:isDark', isDark.value ? '1' : '0');
  },
  {
    immediate: true,
  },
);

watch(
  () => [hours.value, minutes.value, seconds.value],
  () => {
    if (showTime.value) {
      showTime.value = false;
    }
  },
);

const time = computed(() => {
  return showTime.value
    ? showSeconds.value
      ? `${hours.value.toString().padStart(2, '0')}:${minutes.value
          .toString()
          .padStart(2, '0')}:${(seconds.value || 0)
          .toString()
          .padStart(2, '0')}`
      : `${hours.value.toString().padStart(2, '0')}:${minutes.value
          .toString()
          .padStart(2, '0')}`
    : showSeconds.value
    ? '--:--:--'
    : '--:--';
});

const generate = () => {
  hours.value = Math.round(Math.random() * 11) + 1;
  minutes.value = Math.round(Math.random() * 60);
  seconds.value = showSeconds.value
    ? Math.round(Math.random() * 60)
    : undefined;
};
</script>


<style lang="scss">
body {
  margin: 0;
  padding: 0;
  font-family: 'Helvetica Neue', Helvetica, Arial, sans-serif;
  font-size: 14px;
  color: #333;
  background-color: #eee;
  overflow: hidden;
  &.dark {
    color: #eee;
    background-color: #242424;
  }
}
* {
  -webkit-tap-highlight-color: transparent;
}
.read-clock {
  display: flex;
  flex-direction: column;
  align-items: center;
  gap: 20px;
  padding-top: 30px;
  .switchers {
    display: flex;
    gap: 20px;
    margin-top: 30px;
  }
  .time {
    font-size: 32px;
    margin-top: 20px;
    font-family: Courier, monospace, 'Courier New';
    user-select: none;
    -webkit-user-select: none;
  }
  .button {
    width: 120px;
    height: 30px;
    line-height: 30px;
    text-align: center;
    border: none;
    border-radius: 25px;
    background-color: #242424;
    color: #eee;
    font-size: 14px;
    cursor: pointer;
    user-select: none;
    -webkit-user-select: none;
    &:active {
      background-color: #333;
    }
  }
}
body.dark {
  .read-clock {
    .button {
      background-color: #fffb00;
      color: #000;
      &:active {
        background-color: #ddd;
      }
    }
  }
}
</style>