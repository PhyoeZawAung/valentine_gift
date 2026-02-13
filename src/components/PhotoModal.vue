<template>
  <div class="modal-shell" :class="{ open: modelValue }" :aria-hidden="!modelValue">
    <article class="modal-card" ref="cardRef">
      <button class="close-btn photo-close-btn" type="button" @click="$emit('update:modelValue', false)">✕</button>
      <div class="modal-image-wrap">
        <div v-if="!imageLoaded" class="modal-image-skeleton" aria-hidden="true"></div>
        <img
          :src="displaySrc"
          :alt="photo?.alt || ''"
          class="modal-image"
          :class="{ loaded: imageLoaded }"
          decoding="async"
          fetchpriority="high"
          @load="onImageLoad"
          @error="onImageLoad"
        />
      </div>
      <div class="modal-body">
        <p class="modal-caption">{{ photo?.caption }}</p>
        <p class="modal-note">{{ photo?.note }}</p>
      </div>
    </article>
  </div>
</template>

<script setup>
import { ref, watch } from 'vue';

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
const displaySrc = ref('');
const imageLoaded = ref(false);

function onImageLoad() {
  imageLoaded.value = true;
}

watch(
  () => [props.modelValue, props.photo?.src, props.photo?.modalSrc],
  ([open, src, modalSrc]) => {
    if (!open) {
      imageLoaded.value = false;
      return;
    }
    imageLoaded.value = false;
    displaySrc.value = modalSrc || src || '';
  },
  { flush: 'sync' }
);
</script>
