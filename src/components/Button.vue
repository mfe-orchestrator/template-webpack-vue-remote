<script setup lang="ts">
import { ref } from 'vue';

type Variant = 'primary' | 'secondary' | 'danger';

const props = withDefaults(
  defineProps<{
    variant?: Variant;
    disabled?: boolean;
  }>(),
  {
    variant: 'primary',
    disabled: false,
  }
);

const clicks = ref(0);

// Indexed from the template, where the compiler widens the prop back to a
// plain string, so the map is keyed by string rather than by Variant.
const colors: Record<string, string> = {
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
