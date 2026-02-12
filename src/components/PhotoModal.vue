<template>
  <div v-if="modelValue" class="modal-shell" @click.self="$emit('update:modelValue', false)">
    <article class="modal-card" ref="cardRef">
      <button class="close-btn" type="button" @click="$emit('update:modelValue', false)">✕</button>
      <img :src="photo?.src" :alt="photo?.alt" class="modal-image" />
      <div class="modal-body">
        <p class="modal-caption">{{ photo?.caption }}</p>
        <p class="modal-note">{{ photo?.note }}</p>
      </div>
    </article>
  </div>
</template>

<script setup>
import { nextTick, ref, watch } from 'vue';
import { gsap } from 'gsap';

const props = defineProps({
  modelValue: {
    type: Boolean,
    required: true
  },
  photo: {
    type: Object,
    default: null
  }
});

defineEmits(['update:modelValue']);

const cardRef = ref(null);

watch(
  () => props.modelValue,
  async (open) => {
    if (!open) return;
    await nextTick();
    if (!cardRef.value) return;
    gsap.fromTo(
      cardRef.value,
      { y: 26, opacity: 0, scale: 0.94 },
      { y: 0, opacity: 1, scale: 1, duration: 0.42, ease: 'power3.out' }
    );
  }
);
</script>
