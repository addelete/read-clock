<template>
  <div class="Switcher" @click="onChange">
    <div class="Switcher__control" :class="{ checked: modelValue }">
      <div class="Switcher__control__track"></div>
      <div class="Switcher__control__thumb"></div>
    </div>
    <div class="Switcher__label">
      <slot></slot>
    </div>
  </div>
</template>

<script lang="ts" setup>
const props = defineProps<{
  modelValue: boolean;
}>();
const emit = defineEmits<{
  (event: 'update:modelValue', value: boolean): void;
}>();
const onChange = () => {
  emit('update:modelValue', !props.modelValue);
};
</script>

<style lang="scss">
.Switcher {
  user-select: none;
  -webkit-user-select: none;
  display: inline-flex;
  align-items: center;
  gap: 4px;
  &__control {
    position: relative;
    width: 40px;
    height: 20px;
    border-radius: 10px;
    background-color: #eee;
    cursor: pointer;
    transition: background-color 0.2s ease-in-out;
    &__track {
      position: absolute;
      top: 0;
      left: 0;
      width: 100%;
      height: 100%;
      border-radius: 10px;
      background-color: #999;
      transition: background-color 0.2s ease-in-out;
    }

    &__thumb {
      position: absolute;
      top: 2px;
      left: 2px;
      width: 16px;
      height: 16px;
      border-radius: 8px;
      background-color: #fff;
      transition: transform 0.2s ease-in-out;
    }
    &.checked &__track {
      background-color: #04b704;
    }
    &.checked &__thumb {
      transform: translateX(20px);
    }
  }
}
</style>