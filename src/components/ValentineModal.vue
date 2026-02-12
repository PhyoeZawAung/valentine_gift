<template>
  <div v-if="modelValue" class="modal-shell" @click.self="$emit('update:modelValue', false)">
    <article class="modal-card valentine" ref="cardRef">
      <span class="valentine-deco deco-left" aria-hidden="true">❤</span>
      <span class="valentine-deco deco-right" aria-hidden="true">💌</span>
      <button class="close-btn" type="button" @click="$emit('update:modelValue', false)">✕</button>
      <p class="modal-eyebrow">A Little Question</p>
      <h3 class="modal-title">Will you be my Valentine?</h3>
      <p class="modal-copy">I already know what my heart is hoping you will say.</p>
      <div class="modal-actions">
        <button class="btn-primary valentine-yes" type="button" @click="selectYes">Yes, of course ❤</button>
        <button class="btn-secondary valentine-always" type="button" @click="selectYes">Always yes</button>
      </div>
      <p v-if="answered" class="answer-copy">Best answer ever. I love you endlessly.</p>
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
  answered: {
    type: Boolean,
    default: false
  }
});

const emit = defineEmits(['update:modelValue', 'answered']);
const cardRef = ref(null);

function selectYes() {
  emit('answered');
}

watch(
  () => props.modelValue,
  async (open) => {
    if (!open) return;
    await nextTick();
    if (!cardRef.value) return;
    gsap.fromTo(
      cardRef.value,
      { y: 24, opacity: 0, scale: 0.95 },
      { y: 0, opacity: 1, scale: 1, duration: 0.45, ease: 'power3.out' }
    );
  }
);
</script>
