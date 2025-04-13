<template>
    <div class="relative">
        <a @click="dropActive = !dropActive">Menu</a>
        <ul class="absolute left-0"
            v-if="dropActive">
            <li>menu 1</li>
            <li>menu 2</li>
            <li>menu 3</li>
        </ul>
    </div>
</template>

<script setup lang="ts">
    const dropActive = ref(false);
</script>