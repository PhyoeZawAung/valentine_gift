<template>
  <div class="app-shell" :class="{ 'gallery-active': currentScreen === 'gallery' || isPhotoModalOpen }">
    <div class="ambient-bg" aria-hidden="true">
      <span
        v-for="item in floatingItems"
        :key="item.id"
        class="ambient-item"
        :class="item.type"
        :style="item.style"
      >{{ item.symbol }}</span>
    </div>
    <div class="top-fall-layer" aria-hidden="true">
      <span
        v-for="item in fallingItems"
        :key="item.id"
        class="fall-icon"
        :class="item.type"
        :style="item.style"
      >{{ item.symbol }}</span>
    </div>
    <header class="topbar">
      <button class="music-toggle" type="button" @click="toggleMusic">
        <span class="dot" :class="isMusicPlaying ? 'on' : 'off'"></span>
        <span class="music-label">Music {{ isMusicPlaying ? 'On' : 'Off' }}</span>
      </button>
    </header>

    <div class="transition-veil" :class="{ show: isTransitioning }"></div>

    <main class="content-wrap">
      <div class="story-progress-slot">
        <div class="story-progress" :class="{ hidden: !showStoryProgress }" aria-label="Story progress">
          <span
            v-for="(step, index) in storyScreens"
            :key="step"
            class="story-dot"
            :class="{ active: index <= currentStoryIndex }"
          ></span>
        </div>
      </div>
      <div class="screen-stage">
        <Transition mode="out-in" @enter="onScreenEnter" @leave="onScreenLeave">
          <section :key="currentScreen" class="screen-card glass">
          <template v-if="currentScreen === 'countdown'">
            <p class="eyebrow">Countdown</p>
            <h1 class="title">Counting down to our day</h1>
            <p class="subtitle">This surprise unlocks automatically on Valentine time.</p>

            <div class="count-grid">
              <div class="count-box">
                <strong>{{ countdown.days }}</strong>
                <span>Days</span>
              </div>
              <div class="count-box">
                <strong>{{ pad(countdown.hours) }}</strong>
                <span>Hours</span>
              </div>
              <div class="count-box">
                <strong>{{ pad(countdown.minutes) }}</strong>
                <span>Minutes</span>
              </div>
              <div class="count-box">
                <strong>{{ pad(countdown.seconds) }}</strong>
                <span>Seconds</span>
              </div>
            </div>
          </template>

          <template v-else-if="currentScreen === 'welcome'">
            <p class="eyebrow">Our Love Story</p>
            <h1 class="title">Happy Valentine's Baby</h1>
            <p class="subtitle"></p>
            <EnvelopeCard
              :opening="envelopeOpening"
              @open="openEnvelope"
            />
          </template>

          <template v-else-if="currentScreen === 'letter'">
            <p class="eyebrow">Love Letter</p>
            <h2 class="title">For you, always</h2>
            <div class="letter-box" :class="{ 'from-envelope': letterFromEnvelopeIntro }">
              <p class="letter-text">
                <span>{{ typedLetter }}</span><span v-if="isTyping" class="cursor"></span>
              </p>
            </div>
            <button class="btn-primary" :disabled="isTyping" type="button" @click="goTo('timeline')">Continue</button>
          </template>

          <template v-else-if="currentScreen === 'timeline'">
            <p class="eyebrow">Our Story</p>
            <h2 class="title">Little moments, big feelings</h2>
            <ul class="timeline-list">
              <li v-for="item in timelineItems" :key="item.date" class="timeline-item">
                <span class="timeline-icon">{{ item.icon }}</span>
                <div>
                  <p class="timeline-date">{{ item.date }}</p>
                  <p class="timeline-title">{{ item.title }}</p>
                  <p class="timeline-copy">{{ item.description }}</p>
                </div>
              </li>
            </ul>
            <button class="btn-primary" type="button" @click="goTo('gallery')">Next</button>
          </template>

          <template v-else-if="currentScreen === 'gallery'">
            <p class="eyebrow">Our Photos</p>
            <h2 class="title">The way I see you</h2>
            <div class="photo-grid">
              <button
                v-for="photo in visibleGalleryPhotos"
                :key="photo.src"
                v-memo="[photo.src]"
                class="photo-card"
                type="button"
                @pointerdown="onPhotoPointerDown($event)"
                @pointermove="onPhotoPointerMove($event)"
                @pointerup="onPhotoPointerUp(photo, $event)"
                @pointercancel="onPhotoPointerCancel"
                @pointerleave="onPhotoPointerLeave($event)"
                @click.prevent
                @keydown.enter.prevent="openPhoto(photo)"
                @keydown.space.prevent="openPhoto(photo)"
              >
                <img :src="photo.src" :alt="photo.alt" loading="lazy" decoding="async" />
              </button>
            </div>
            <div ref="galleryLoadSentinel" class="gallery-sentinel" aria-hidden="true"></div>
            <button class="btn-primary" type="button" @click="goTo('final')">Final surprise</button>
          </template>

          <template v-else>
            <p class="eyebrow">From Me To You</p>
            <h2 class="title">Every day with you is my favorite day</h2>
            <p class="subtitle">We have shared <strong>{{ daysTogether }}</strong> days. I would choose every one again.</p>
            <button class="btn-primary" type="button" @click="openValentineModal">One last thing...</button>
          </template>
          </section>
        </Transition>
      </div>
    </main>

    <PhotoModal v-model="isPhotoModalOpen" :photo="selectedPhoto" />
    <ValentineModal
      v-model="isValentineModalOpen"
      :answered="finalMessage"
      @answered="onValentineAnswered"
    />

    <audio ref="bgMusic" src="/assets/music/music.mp3" preload="auto" loop></audio>
  </div>
</template>

<script setup>
import { computed, nextTick, onBeforeUnmount, onMounted, ref } from 'vue';
import { gsap } from 'gsap';
import EnvelopeCard from './components/EnvelopeCard.vue';
import PhotoModal from './components/PhotoModal.vue';
import ValentineModal from './components/ValentineModal.vue';
import { galleryPhotos, letterText, timelineItems } from './data/content';

const currentScreen = ref('countdown');
const isTransitioning = ref(false);
const envelopeOpening = ref(false);
const letterFromEnvelopeIntro = ref(false);
const countdown = ref({ days: 0, hours: 0, minutes: 0, seconds: 0 });
const typedLetter = ref('');
const isTyping = ref(false);
const isMusicPlaying = ref(false);
const finalMessage = ref(false);
const floatingItems = ref([]);
const fallingItems = ref([]);
const isPhotoModalOpen = ref(false);
const isValentineModalOpen = ref(false);
const selectedPhoto = ref(null);
const bgMusic = ref(null);
const transitionFromScreen = ref('');
const transitionToScreen = ref('');
const storyScreens = ['welcome', 'letter', 'timeline', 'gallery', 'final'];
const galleryLoadSentinel = ref(null);
const galleryBatchSize = 18;
const galleryVisibleCount = ref(galleryBatchSize);
const PHOTO_CACHE_KEY = 'valentine_photo_cache_v1';
const photoLocalCache = ref({});
const photoTap = ref({
  activePointerId: null,
  startX: 0,
  startY: 0,
  startTs: 0,
  moved: false
});
const lastGalleryScrollAt = ref(0);
let galleryObserver = null;
let preloadIdleId = null;
let preloadTimeoutId = null;

let countdownTimer = null;
let typingTimer = null;

const anniversaryDate = new Date('2025-06-14T00:00:00');
const unlockDate = getNextValentineDate();

const daysTogether = computed(() => {
  const diff = Date.now() - anniversaryDate.getTime();
  return Math.max(1, Math.floor(diff / 86400000)).toLocaleString();
});
const currentStoryIndex = computed(() => storyScreens.indexOf(currentScreen.value));
const showStoryProgress = computed(() => currentStoryIndex.value >= 0);
const visibleGalleryPhotos = computed(() => galleryPhotos.slice(0, galleryVisibleCount.value));

function getNextValentineDate() {
  const now = new Date();
  const target = new Date(now.getFullYear(), 1, 10, 0, 0, 0);
  // if (now > target) {
  //   return new Date(now.getFullYear() + 1, 1, 14, 0, 0, 0);
  // }
  return target;
}

function pad(value) {
  return String(value).padStart(2, '0');
}

function computeCountdown() {
  const diff = unlockDate.getTime() - Date.now();
  if (diff <= 0) {
    if (countdownTimer) {
      clearInterval(countdownTimer);
      countdownTimer = null;
    }
    countdown.value = { days: 0, hours: 0, minutes: 0, seconds: 0 };
    goTo('welcome');
    return;
  }

  const totalSeconds = Math.floor(diff / 1000);
  countdown.value = {
    days: Math.floor(totalSeconds / 86400),
    hours: Math.floor((totalSeconds % 86400) / 3600),
    minutes: Math.floor((totalSeconds % 3600) / 60),
    seconds: totalSeconds % 60
  };
}

function startCountdown() {
  computeCountdown();
  if (Date.now() >= unlockDate.getTime()) {
    currentScreen.value = 'welcome';
    return;
  }
  countdownTimer = setInterval(computeCountdown, 1000);
}

function goTo(screen, options = {}) {
  if (currentScreen.value === screen) return;
  const fromEnvelope = Boolean(options.fromEnvelope);
  const fromGalleryToFinal = currentScreen.value === 'gallery' && screen === 'final';
  transitionFromScreen.value = currentScreen.value;
  transitionToScreen.value = screen;
  isTransitioning.value = !fromEnvelope;

  // Stop gallery background work immediately before leaving gallery
  if (currentScreen.value === 'gallery' && galleryObserver) {
    galleryObserver.disconnect();
    galleryObserver = null;
    clearGalleryPreloadSchedule();
  }

  setTimeout(() => {
    currentScreen.value = screen;
    if (screen === 'letter') {
      // Keep envelope->letter clean: no extra paper-card entrance flash.
      letterFromEnvelopeIntro.value = false;
      setTimeout(() => {
        startTypewriter();
      }, 120);
    }
    if (screen === 'gallery') {
      nextTick(() => {
        setupGalleryObserver();
      });
    } else if (fromGalleryToFinal) {
      // Keep next visits smooth by reducing gallery DOM after transition
      setTimeout(() => {
        galleryVisibleCount.value = galleryBatchSize;
      }, 120);
    }
    setTimeout(() => {
      isTransitioning.value = false;
    }, fromEnvelope ? 80 : 300);
  }, 140);
}
function openEnvelope() {
  if (envelopeOpening.value) return;
  envelopeOpening.value = true;
  setTimeout(() => {
    goTo('letter', { fromEnvelope: true });
  }, 320);
  setTimeout(() => {
    envelopeOpening.value = false;
  }, 520);
}

function startTypewriter() {
  if (typingTimer) {
    clearTimeout(typingTimer);
    typingTimer = null;
  }
  typedLetter.value = '';
  isTyping.value = true;
  let idx = 0;

      const typeTick = () => {
        if (idx <= letterText.length) {
          typedLetter.value = letterText.slice(0, idx);
          idx += 1;
          typingTimer = setTimeout(typeTick, 92);
          return;
        }
        isTyping.value = false;
      };

  typeTick();
}

function openPhoto(photo) {
  if (!photo || hasRecentGalleryScroll(140)) return;
  if (isPhotoModalOpen.value && selectedPhoto.value?.src === photo.src) return;
  const cachedSrc = photoLocalCache.value[photo.src];
  selectedPhoto.value = {
    ...photo,
    modalSrc: cachedSrc || photo.src
  };
  isPhotoModalOpen.value = true;
  preloadPhotoDecode(photo.src);

  if (!cachedSrc) {
    cachePhotoForModal(photo.src).then((dataUrl) => {
      if (!dataUrl) return;
      if (
        isPhotoModalOpen.value &&
        selectedPhoto.value &&
        selectedPhoto.value.src === photo.src
      ) {
        selectedPhoto.value = {
          ...selectedPhoto.value,
          modalSrc: dataUrl
        };
      }
    });
  }
}

function onPhotoPointerDown(event) {
  if (hasRecentGalleryScroll(120)) return;
  photoTap.value = {
    activePointerId: event.pointerId,
    startX: event.clientX,
    startY: event.clientY,
    startTs: performance.now(),
    moved: false
  };
}

function onPhotoPointerMove(event) {
  if (photoTap.value.activePointerId !== event.pointerId) return;
  const dx = Math.abs(event.clientX - photoTap.value.startX);
  const dy = Math.abs(event.clientY - photoTap.value.startY);
  if (dx > 10 || dy > 10) {
    photoTap.value.moved = true;
  }
}

function onPhotoPointerLeave(event) {
  if (photoTap.value.activePointerId !== event.pointerId) return;
  photoTap.value.moved = true;
}

function onPhotoPointerCancel() {
  photoTap.value.activePointerId = null;
  photoTap.value.moved = true;
}

function onPhotoPointerUp(photo, event) {
  if (photoTap.value.activePointerId !== event.pointerId) return;

  const elapsed = performance.now() - photoTap.value.startTs;
  const isTap = !photoTap.value.moved && elapsed < 320 && !hasRecentGalleryScroll(150);

  photoTap.value.activePointerId = null;
  photoTap.value.moved = false;

  if (isTap) {
    openPhoto(photo);
  }
}

function openValentineModal() {
  isValentineModalOpen.value = true;
}

function onValentineAnswered() {
  finalMessage.value = true;
  dropHearts();
}

function buildFloatingItems() {
  const symbols = ['❤', '💕', '🌸', '🍓', '💗'];
  const total = 28;
  const items = [];

  for (let i = 0; i < total; i += 1) {
    const symbol = symbols[Math.floor(Math.random() * symbols.length)];
    const type = symbol === '🍓' ? 'berry' : symbol === '🌸' ? 'blossom' : 'heart';
    items.push({
      id: `${i}-${Date.now()}`,
      symbol,
      type,
      style: {
        left: `${Math.random() * 100}%`,
        top: `${Math.random() * 100}%`,
        fontSize: `${0.8 + Math.random() * 1.2}rem`,
        animationDuration: `${10 + Math.random() * 16}s`,
        animationDelay: `${Math.random() * 8}s`
      }
    });
  }

  floatingItems.value = items;
}

function buildFallingItems() {
  const symbols = ['❤', '🌸', '🍓', '💗', '✨'];
  const total = 16;
  const items = [];

  for (let i = 0; i < total; i += 1) {
    const symbol = symbols[Math.floor(Math.random() * symbols.length)];
    const type = symbol === '🍓' ? 'berry' : symbol === '🌸' ? 'blossom' : 'heart';
    items.push({
      id: `fall-${i}-${Date.now()}`,
      symbol,
      type,
      style: {
        left: `${Math.random() * 100}%`,
        animationDuration: `${9 + Math.random() * 8}s`,
        animationDelay: `${Math.random() * 9}s`,
        fontSize: `${0.9 + Math.random() * 1.2}rem`
      }
    });
  }

  fallingItems.value = items;
}

function loadMoreGalleryItems() {
  if (galleryVisibleCount.value >= galleryPhotos.length) return;
  galleryVisibleCount.value = Math.min(
    galleryPhotos.length,
    galleryVisibleCount.value + galleryBatchSize
  );
  scheduleGalleryPreload();
}

function setupGalleryObserver() {
  if (galleryObserver) {
    galleryObserver.disconnect();
    galleryObserver = null;
  }
  if (!galleryLoadSentinel.value) return;

  galleryObserver = new IntersectionObserver(
    (entries) => {
      entries.forEach((entry) => {
        if (entry.isIntersecting) {
          loadMoreGalleryItems();
        }
      });
    },
    { rootMargin: '180px 0px 180px 0px' }
  );
  galleryObserver.observe(galleryLoadSentinel.value);
  scheduleGalleryPreload();
}

function onWindowScroll() {
  if (currentScreen.value !== 'gallery') return;
  lastGalleryScrollAt.value = performance.now();
}

function hasRecentGalleryScroll(windowMs) {
  return performance.now() - lastGalleryScrollAt.value < windowMs;
}

function clearGalleryPreloadSchedule() {
  if (preloadIdleId !== null && 'cancelIdleCallback' in window) {
    window.cancelIdleCallback(preloadIdleId);
    preloadIdleId = null;
  }
  if (preloadTimeoutId !== null) {
    clearTimeout(preloadTimeoutId);
    preloadTimeoutId = null;
  }
}

function preloadPhotoDecode(src) {
  if (!src) return;
  const img = new Image();
  img.decoding = 'async';
  img.src = src;
  if (typeof img.decode === 'function') {
    img.decode().catch(() => {});
  }
}

function warmupVisibleGalleryPhotos() {
  const upperBound = Math.min(galleryVisibleCount.value + 10, galleryPhotos.length);
  for (let index = 0; index < upperBound; index += 1) {
    preloadPhotoDecode(galleryPhotos[index].src);
  }
}

function scheduleGalleryPreload() {
  if (currentScreen.value !== 'gallery') return;
  clearGalleryPreloadSchedule();

  if ('requestIdleCallback' in window) {
    preloadIdleId = window.requestIdleCallback(
      () => {
        preloadIdleId = null;
        warmupVisibleGalleryPhotos();
      },
      { timeout: 800 }
    );
    return;
  }

  preloadTimeoutId = setTimeout(() => {
    preloadTimeoutId = null;
    warmupVisibleGalleryPhotos();
  }, 140);
}

function loadPhotoCacheFromStorage() {
  try {
    const raw = localStorage.getItem(PHOTO_CACHE_KEY);
    if (!raw) return;
    const parsed = JSON.parse(raw);
    if (parsed && typeof parsed === 'object') {
      photoLocalCache.value = parsed;
    }
  } catch (_) {
    photoLocalCache.value = {};
  }
}

function persistPhotoCache() {
  try {
    localStorage.setItem(PHOTO_CACHE_KEY, JSON.stringify(photoLocalCache.value));
  } catch (_) {
    // Ignore quota/storage failures and continue with network fallback.
  }
}

function blobToDataUrl(blob) {
  return new Promise((resolve, reject) => {
    const reader = new FileReader();
    reader.onload = () => resolve(String(reader.result || ''));
    reader.onerror = reject;
    reader.readAsDataURL(blob);
  });
}

async function cachePhotoForModal(src) {
  if (!src || photoLocalCache.value[src]) {
    return photoLocalCache.value[src] || '';
  }

  try {
    const response = await fetch(src, { cache: 'force-cache' });
    if (!response.ok) return '';
    const blob = await response.blob();

    // Keep localStorage usage conservative on mobile browsers.
    if (blob.size > 360 * 1024) return '';

    const dataUrl = await blobToDataUrl(blob);
    if (!dataUrl || dataUrl.length > 520 * 1024) return '';

    photoLocalCache.value = {
      ...photoLocalCache.value,
      [src]: dataUrl
    };
    persistPhotoCache();
    return dataUrl;
  } catch (_) {
    return '';
  }
}

function dropHearts() {
  const layer = document.createElement('div');
  layer.className = 'heart-rain';
  document.body.appendChild(layer);

  for (let i = 0; i < 46; i += 1) {
    const node = document.createElement('span');
    node.className = 'rain-item';
    node.textContent = Math.random() > 0.32 ? '❤' : '🌸';
    node.style.left = `${Math.random() * 100}vw`;
    node.style.animationDuration = `${3.5 + Math.random() * 3.2}s`;
    node.style.animationDelay = `${Math.random() * 1.1}s`;
    layer.appendChild(node);
  }

  setTimeout(() => {
    layer.remove();
  }, 9000);
}

function onScreenEnter(el, done) {
  const isEnvelopeToLetter =
    transitionFromScreen.value === 'welcome' &&
    transitionToScreen.value === 'letter';
  const isGalleryToFinal =
    transitionFromScreen.value === 'gallery' &&
    transitionToScreen.value === 'final';

  if (isEnvelopeToLetter) {
    gsap.fromTo(
      el,
      { opacity: 0 },
      {
        opacity: 1,
        duration: 0.55,
        ease: 'power2.out',
        onComplete: () => {
          transitionFromScreen.value = '';
          transitionToScreen.value = '';
          done();
        }
      }
    );
    return;
  }

  if (isGalleryToFinal) {
    gsap.fromTo(
      el,
      { opacity: 0.92 },
      {
        opacity: 1,
        duration: 0.24,
        ease: 'power1.out',
        onComplete: () => {
          transitionFromScreen.value = '';
          transitionToScreen.value = '';
          done();
        }
      }
    );
    return;
  }

  gsap.fromTo(
    el,
    { y: 18, opacity: 0, scale: 0.985, filter: 'blur(8px)' },
    {
      y: 0,
      opacity: 1,
      scale: 1,
      filter: 'blur(0px)',
      duration: 0.62,
      ease: 'power3.out',
      onComplete: done
    }
  );
}

function onScreenLeave(el, done) {
  const isEnvelopeToLetter =
    transitionFromScreen.value === 'welcome' &&
    transitionToScreen.value === 'letter';
  const isGalleryToFinal =
    transitionFromScreen.value === 'gallery' &&
    transitionToScreen.value === 'final';

  if (isEnvelopeToLetter) {
    gsap.to(el, {
      opacity: 0,
      duration: 0.34,
      ease: 'power2.inOut',
      onComplete: done
    });
    return;
  }

  if (isGalleryToFinal) {
    gsap.to(el, {
      opacity: 0,
      duration: 0.18,
      ease: 'power1.inOut',
      onComplete: done
    });
    return;
  }

  gsap.to(el, {
    y: -10,
    opacity: 0,
    scale: 1.01,
    filter: 'blur(8px)',
    duration: 0.42,
    ease: 'power2.inOut',
    onComplete: done
  });
}

async function tryAutoplayMusic() {
  const audio = bgMusic.value;
  if (!audio) return;
  try {
    audio.volume = 0.72;
    await audio.play();
    isMusicPlaying.value = true;
  } catch (_) {
    isMusicPlaying.value = false;
    bindFirstInteractionMusic();
  }
}

function bindFirstInteractionMusic() {
  const attempt = async () => {
    window.removeEventListener('pointerdown', attempt);
    window.removeEventListener('touchstart', attempt);
    window.removeEventListener('keydown', attempt);
    await tryAutoplayMusic();
  };

  window.addEventListener('pointerdown', attempt, { once: true });
  window.addEventListener('touchstart', attempt, { once: true });
  window.addEventListener('keydown', attempt, { once: true });
}

async function toggleMusic() {
  const audio = bgMusic.value;
  if (!audio) return;
  if (audio.paused) {
    try {
      await audio.play();
      isMusicPlaying.value = true;
    } catch (_) {
      isMusicPlaying.value = false;
    }
  } else {
    audio.pause();
    isMusicPlaying.value = false;
  }
}

onMounted(() => {
  loadPhotoCacheFromStorage();
  buildFloatingItems();
  buildFallingItems();
  startCountdown();
  tryAutoplayMusic();
  window.addEventListener('scroll', onWindowScroll, { passive: true });
});

onBeforeUnmount(() => {
  if (countdownTimer) clearInterval(countdownTimer);
  if (typingTimer) clearTimeout(typingTimer);
  if (galleryObserver) galleryObserver.disconnect();
  clearGalleryPreloadSchedule();
  window.removeEventListener('scroll', onWindowScroll);
});
</script>
