<template>
  <button class="envelope-btn" type="button" @click="$emit('open', $event)" aria-label="Open envelope">
    <div class="envelope" :class="{ opening }">
      <div class="envelope-back"></div>
      <div class="envelope-letter">For my favorite person</div>
      <div class="envelope-flap"></div>
      <div class="envelope-front"></div>
      <div class="seal-heart" aria-hidden="true">❤</div>
    </div>
  </button>
</template>

<script setup>
defineProps({
  opening: {
    type: Boolean,
    default: false
  }
});

defineEmits(['open']);
</script>
