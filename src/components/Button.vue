<script setup>
import { ref } from 'vue';

const props = defineProps({
  variant: {
    type: String,
    default: 'primary',
  },
  disabled: {
    type: Boolean,
    default: false,
  },
});

const clicks = ref(0);

const colors = {
  primary: '#007bff',
  secondary: '#6c757d',
  danger: '#dc3545',
};
</script>

<template>
  <button
    id="click-btn"
    :disabled="props.disabled"
    :style="{
      padding: '10px 20px',
      backgroundColor: props.disabled ? '#cccccc' : colors[props.variant] || colors.primary,
      color: '#ffffff',
      border: 'none',
      borderRadius: '4px',
      cursor: props.disabled ? 'not-allowed' : 'pointer',
      fontSize: '16px',
      fontWeight: '500',
      opacity: props.disabled ? 0.6 : 1,
    }"
    @click="clicks++"
  >
    <slot>Click me: {{ clicks }}</slot>
  </button>
</template>
