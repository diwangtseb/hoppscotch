<template>
  <div
    class="
      cursor-pointer
      flex
      py-2
      px-6
      transition
      items-center
      group
      hover:bg-primaryLight
      focus:outline-none
      focus-visible:bg-primaryLight
    "
    tabindex="0"
    @click="$emit('action', shortcut.action)"
    @keydown.enter="$emit('action', shortcut.action)"
  >
    <SmartIcon
      class="
        mr-4
        opacity-75
        transition
        svg-icons
        group-hover:opacity-100
        group-focus:opacity-100
      "
      :name="shortcut.icon"
    />
    <span
      class="
        flex flex-1
        mr-4
        transition
        group-hover:text-secondaryDark
        group-focus:text-secondaryDark
      "
    >
      {{ $t(shortcut.label) }}
    </span>
    <span
      v-for="(key, keyIndex) in shortcut.keys"
      :key="`key-${String(keyIndex)}`"
      class="shortcut-key"
    >
      {{ key }}
    </span>
  </div>
</template>

<script setup lang="ts">
defineProps<{
  shortcut: Object
}>()
</script>

<style lang="scss" scoped>
.shortcut-key {
  @apply bg-dividerLight;
  @apply rounded;
  @apply ml-2;
  @apply py-1;
  @apply px-2;
  @apply inline-flex;
}
</style>
