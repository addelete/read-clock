

<template>
  <v-stage :config="configKonva">
    <v-layer>
      <v-text v-if="!isPhone" :config="logoText" @click="goToGithub" />
    </v-layer>
    <v-layer
      :offset="{
        x: -props.size / 2,
        y: -props.size / 2,
      }"
    >
      <v-group>
        <v-rect
          v-for="(item, index) in scaleList"
          :key="index"
          :config="item"
        />
        <template v-if="displayNumbersLevel > 0">
          <v-text
            v-for="(item, index) in hourNumberList"
            :key="index"
            :config="item"
          />
        </template>
        <template v-if="displayNumbersLevel > 1">
          <v-text
            v-for="(item, index) in minuteNumberList"
            :key="index"
            :config="item"
          />
        </template>
      </v-group>
      <v-group>
        <v-circle :config="bgCenterPoint" />
      </v-group>
      <v-group
        v-if="showHour && hourHandConfig"
        :offset="hourHandConfig.offset"
        :rotation="hourHandRotation"
        @pointerdown="onPointDownSecond($event, 'hour')"
        ref="hourHandRef"
      >
        <v-rect :config="hourHandConfig.touchArea" />
        <v-rect :config="hourHandConfig.hand" />
      </v-group>
      <v-group
        v-if="showMinute && minuteHandConfig"
        :offset="minuteHandConfig.offset"
        :rotation="minuteHandRotation"
        @pointerdown="onPointDownSecond($event, 'minute')"
        ref="minuteHandRef"
      >
        <v-rect :config="minuteHandConfig.touchArea" />
        <v-rect :config="minuteHandConfig.hand" />
      </v-group>
      <v-group
        v-if="showSecond && secondHandConfig"
        :offset="secondHandConfig.offset"
        :rotation="secondHandRotation"
        @pointerdown="onPointDownSecond($event, 'second')"
        ref="secondHandRef"
      >
        <v-rect :config="secondHandConfig.touchArea" />
        <v-rect :config="secondHandConfig.hand" />
      </v-group>
      <v-group>
        <v-circle :config="centerPoint" />
      </v-group>
    </v-layer>
  </v-stage>
</template>

<script lang="ts" setup>
import Konva from 'konva';
import { KonvaEventObject } from 'konva/lib/Node';
import { computed, ref, watch } from 'vue';

const props = withDefaults(
  defineProps<{
    hours?: number; // 0-23
    minutes?: number; // 0-59
    seconds?: number; // 0-59
    size?: number;
    isDark?: boolean;
    displayNumbersLevel: number; // 0-3
  }>(),
  {
    size: 500,
    isDark: false,
    displayNumbersLevel: 1,
  },
);

const emit = defineEmits<{
  (event: 'update:hours', val: number): void;
  (event: 'update:minutes', val: number): void;
  (event: 'update:seconds', val: number): void;
}>();

const isPhone = /mobile/i.test(navigator.userAgent);

const centerPageX = ref(0); // 拨动指针时的表盘中心位置
const centerPageY = ref(0); // 拨动指针时的表盘中心位置
const movingHandType = ref(''); // 拨动指针时的指针类型
const hourHandRef = ref<any>(); // 秒针ref
const minuteHandRef = ref<any>(); // 秒针ref
const secondHandRef = ref<any>(); // 秒针ref
let minuteHandOffsetR = 0; // 指针拨动时的秒数造成的分钟指针偏移
let hourHandOffsetR = 0; // 指针拨动时的分钟数造成的时钟指针偏移

const hourHandRotation = ref<number>();
const minuteHandRotation = ref<number>();
const secondHandRotation = ref<number>();

const color = computed(() => {
  return props.isDark ? '#eee' : '#242424';
});

const genHand = (w: number, h: number, o = 1, c = color.value) => {
  const width = (w * props.size) / 500;
  const height = (props.size / 2) * h;
  const touchAreaWidth = (30 * props.size) / 500;
  return {
    offset: {
      x: touchAreaWidth / 2,
      y: height * o - width / 2,
    },
    touchArea: {
      fill: 'red',
      width: touchAreaWidth,
      height,
      opacity: 0,
    },
    hand: {
      fill: c,
      width,
      height,
      offsetX: (width - touchAreaWidth) / 2,
      cornerRadius: width / 2,
      shadowColor: 'black',
      shadowBlur: width,
      shadowOpacity: 0.2,
    },
  };
};

// 时针角度
watch(
  [() => props.hours, () => props.minutes, () => movingHandType.value],
  () => {
    if (movingHandType.value !== '' || props.hours === undefined) {
      return;
    } else {
      hourHandRotation.value =
        (props.hours || 0) * 30 + ((props.minutes || 0) * 6 || 0) / 12;
    }
  },
  {
    immediate: true,
  },
);

// 分针角度
watch(
  [() => props.minutes, () => props.seconds, () => movingHandType.value],
  () => {
    if (movingHandType.value !== '' || props.minutes === undefined) {
      return;
    } else {
      minuteHandRotation.value =
        (props.minutes || 0) * 6 + ((props.seconds || 0) * 6 || 0) / 60;
    }
  },
  {
    immediate: true,
  },
);

// 秒针角度
watch(
  [() => props.seconds, () => movingHandType.value],
  () => {
    if (movingHandType.value !== '' || props.seconds === undefined) {
      return;
    } else {
      secondHandRotation.value = (props.seconds || 0) * 6;
    }
  },
  {
    immediate: true,
  },
);

// 显示时针
const showHour = computed(() => {
  return props.hours !== undefined;
});
// 显示分针
const showMinute = computed(() => {
  return props.minutes !== undefined;
});
// 显示秒针
const showSecond = computed(() => {
  return props.seconds !== undefined;
});

// 画板配置
const configKonva = computed(() => {
  return {
    width: props.size,
    height: props.size,
  };
});

const logoText = computed(() => {
  return {
    x: 0,
    y: scaleNumber(100),
    fill: color.value,
    text: 'github',
    fontSize: scaleNumber(20),
    fontFamily: 'serif',
    fontStyle: 'italic',
    width: props.size,
    align: 'center',
    opacity: 0.5,
  };
});

// 刻度线设置列表
const scaleList = computed(() => {
  const list = [];
  const bigWidth = scaleNumber(8);
  const smallWidth = scaleNumber(2);
  const bigHeight =
    props.displayNumbersLevel > 1
      ? scaleNumber(10)
      : props.displayNumbersLevel > 0
      ? scaleNumber(20)
      : scaleNumber(30);
  const smallHeight =
    props.displayNumbersLevel > 1
      ? scaleNumber(8)
      : props.displayNumbersLevel > 0
      ? scaleNumber(10)
      : scaleNumber(15);
  for (let i = 0; i < 60; i++) {
    const isBig = i % 5 === 0;
    const width = isBig ? bigWidth : smallWidth;
    const height = isBig ? bigHeight : smallHeight;
    const rangle = ((i - 15) * 6 * Math.PI) / 180;
    // 水平偏移
    const cosScale = Math.cos(rangle);
    const sinScale = Math.sin(rangle);
    const radius = props.size / 2 - (props.displayNumbersLevel > 1 ? scaleNumber(15) : 0);
    const offsetX = cosScale * radius + (width / 2) * sinScale;
    // 垂直偏移
    const offsetY = sinScale * radius - (width / 2) * cosScale;
    // 加入刻度线列表
    list.push({
      x: offsetX,
      y: offsetY,
      fill: color.value,
      width,
      height,
      rotation: i * 6,
    });
  }
  return list;
});

// 时数字设置列表
const hourNumberList = computed(() => {
  const list = [];
  const fontSize = scaleNumber(60);
  const centerX = -fontSize;
  const centerY = -fontSize * 0.5;
  for (let i = 1; i <= 12; i++) {
    // 水平偏移
    const offsetX =
      Math.cos(((i - 3) * 30 * Math.PI) / 180) * (props.size / 2 - fontSize);
    // 垂直偏移
    const offsetY =
      Math.sin(((i - 3) * 30 * Math.PI) / 180) * (props.size / 2 - fontSize);
    // 加入数字列表
    list.push({
      x: centerX + offsetX,
      y: centerY + offsetY,
      fill: color.value,
      text: i,
      fontSize: fontSize,
      fontStyle: 'bold',
      width: fontSize * 2,
      align: 'center',
      verticalAlign: 'middle',
      letterSpacing: -fontSize * 0.15,
      opacity: 0.8,
    });
  }
  return list;
});

// 分刻数字设置列表
const minuteNumberList = computed(() => {
  const list = [];
  const fontSize = scaleNumber(12);
  const centerX = -fontSize;
  const centerY = -fontSize * 0.5;
  for (let i = 1; i <= 60; i++) {
    if (props.displayNumbersLevel < 2) continue;
    if (i % 5 !== 0 && props.displayNumbersLevel < 3) continue;
    // 水平偏移
    const offsetX =
      Math.cos(((i - 15) * 6 * Math.PI) / 180) *
      (props.size / 2 - fontSize / 2);
    // 垂直偏移
    const offsetY =
      Math.sin(((i - 15) * 6 * Math.PI) / 180) *
      (props.size / 2 - fontSize / 2);
    // 加入数字列表
    list.push({
      x: centerX + offsetX,
      y: centerY + offsetY,
      fill: color.value,
      text: i.toString().padStart(2, '0'),
      fontSize: fontSize,
      width: fontSize * 2,
      align: 'center',
      verticalAlign: 'middle',
      opacity: 0.8,
    });
  }
  return list;
});

const bgCenterPoint = computed(() => {
  return {
    radius: scaleNumber(12),
    fill: color.value,
  };
});
// 时针配置
const hourHandConfig = computed(() => {
  return genHand(12, 0.7);
});
// 分针配置
const minuteHandConfig = computed(() => {
  return genHand(8, 0.9);
});
// 秒针配置
const secondHandConfig = computed(() => {
  return genHand(2, 1.05, 0.9, '#EA6040');
});
const centerPoint = computed(() => {
  const radius = scaleNumber(3);
  return {
    fill: color.value,
    stroke: '#EA6040',
    strokeWidth: 2,
    radius,
  };
});

const testClockwise = (start: number, end: number) => {
  start = (start + 360) % 360;
  end = (end + 360) % 360;
  const astrideCircle = Math.abs(start - end) > 180;
  const clockwise =
    (start < end && end - start < 180) || (start > end && astrideCircle);
  return { clockwise, astrideCircle };
};

const onPointMove = (e: any) => {
  let pageX = 0;
  let pageY = 0;
  if (isPhone) {
    pageX = e.touches[0].pageX;
    pageY = e.touches[0].pageY;
  } else {
    pageX = e.pageX;
    pageY = e.pageY;
  }
  const currentR =
    (Math.atan2(pageY - centerPageY.value, pageX - centerPageX.value) * 180) /
      Math.PI +
    90;

  const node = (
    {
      hour: hourHandRef,
      minute: minuteHandRef,
      second: secondHandRef,
    }[movingHandType.value] as any
  ).value.getNode() as Konva.Group;
  const oldR = node.rotation();
  const { clockwise, astrideCircle } = testClockwise(oldR, currentR);
  node.rotate(currentR - oldR);
  switch (movingHandType.value) {
    case 'hour':
      emit('update:hours', ((Math.round(currentR / 30) + 11) % 12) + 1);
      break;
    case 'minute':
      emit('update:minutes', (Math.round(currentR / 6) + 60) % 60);
      if (astrideCircle) {
        const hours = (props.hours || 0) + (clockwise ? 1 : -1);
        emit('update:hours', ((hours + 11) % 12) + 1);
        hourHandRef.value
          ?.getNode()
          ?.rotate(
            clockwise
              ? 30 - hourHandOffsetR
              : hourHandOffsetR
              ? -hourHandOffsetR
              : -30,
          );
        hourHandOffsetR = 0;
      }

      break;
    case 'second':
      emit('update:seconds', (Math.round(currentR / 6) + 60) % 60);
      if (astrideCircle) {
        const minutes = (props.minutes || 0) + (clockwise ? 1 : -1);
        emit('update:minutes', (minutes + 60) % 60);
        if (minutes >= 60 || minutes < 0) {
          emit(
            'update:hours',
            (((props.hours || 0) + (clockwise ? 1 : -1) + 11) % 12) + 1,
          );
        }
        minuteHandRef.value
          ?.getNode()
          ?.rotate(
            clockwise
              ? 6 - minuteHandOffsetR
              : minuteHandOffsetR
              ? -minuteHandOffsetR
              : -6,
          );
        hourHandRef.value?.getNode()?.rotate(clockwise ? 0.5 : -0.5);
        minuteHandOffsetR = 0;
      }

      break;
    default:
      break;
  }
};

const onPointUp = () => {
  if (isPhone) {
    document.removeEventListener('touchmove', onPointMove);
    document.removeEventListener('touchend', onPointUp);
  } else {
    document.removeEventListener('mousemove', onPointMove);
    document.removeEventListener('mouseup', onPointUp);
  }
  movingHandType.value = '';
  minuteHandOffsetR = 0;
  hourHandOffsetR = 0;
};

const onPointDownSecond = (event: KonvaEventObject<any>, hand: string) => {
  event.cancelBubble = true;
  const centerX = props.size / 2;
  const centerY = props.size / 2;
  centerPageX.value = event.evt.pageX + (centerX - event.evt.offsetX);
  centerPageY.value = event.evt.pageY + (centerY - event.evt.offsetY);
  movingHandType.value = hand;
  minuteHandOffsetR = ((props.seconds || 0) / 60) * 6;
  hourHandOffsetR = ((props.minutes || 0) / 60) * 30;

  if (isPhone) {
    document.addEventListener('touchmove', onPointMove);
    document.addEventListener('touchend', onPointUp);
  } else {
    document.addEventListener('mousemove', onPointMove);
    document.addEventListener('mouseup', onPointUp);
  }
};

const scaleNumber = (number: number) => {
  return (props.size / 500) * number;
};

const goToGithub = () => {
  window.location.href = 'https://github.com/addelete/read-clock'
}
</script>

<style lang="scss">
</style>