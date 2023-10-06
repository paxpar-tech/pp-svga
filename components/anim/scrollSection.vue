<script setup lang="ts"></script>

<template>
  <div class="scroll-section scroll-section-v300">
    <div class="sticky">
      <slot />
    </div>
  </div>
</template>

<style>
div.sticky {
  top: 50%;
  transform: translate(0%, -50%);
}

.scroll-section {
  /* background-color: rgb(1, 113, 135); */
  height: 100vh;
  /* background-color: green; */
}

.scroll-section-v100 {
  height: 100vh;
}

.scroll-section-v300 {
  height: 300vh;
}

.noscroll-section {
  height: 100vh;
  /* background-color: green; */
}
</style>
