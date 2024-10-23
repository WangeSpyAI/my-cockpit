<template>
  <button
    class="relative flex items-center justify-center w-32 p-1 rounded-md shadow-inner h-9 bg-slate-800/60"
    @click="toggleFilter"
  >
    <div
      class="absolute top-auto flex items-center px-1 rounded-[4px] shadow transition-all w-[70%] h-[80%]"
      :class="
        isFilterOn === undefined
          ? 'justify-start bg-slate-800/60 text-slate-400 left-[4%]'
          : isFilterOn
          ? 'bg-red-700 hover:bg-red-800 text-slate-50 justify-end left-[26%]'
          : 'bg-green-700 hover:bg-green-800 text-slate-400 justify-start left-[4%]'
      "
    >
      <span class="inline-block font-extrabold align-middle">
        <!-- {{ isFilterOn ? 'FILTER OFF' : 'FILTER ON' }} -->
        {{ isFilterOn }}
      </span>
    </div>
  </button>
</template>

<script setup lang="ts">

import { ref, inject } from 'vue';

const isFilterOn = inject('isFilterOn', ref(false));
const toggleFilter = () => {
  isFilterOn.value = !isFilterOn.value;
};
</script>
