<template>
    <div class="bg-theme">
        <div class="container mx-auto px-4">
            <header class="flex justify-between py-4">
                <div>
                    <h4><NuxtLink to="/">Neil Axinto</NuxtLink></h4>
                </div>
                <div>
                    <ul class="flex gap-x-7">
                        <li>menu 1</li>
                        <li>menu 2</li>
                        <li>menu 3</li>
                    </ul>
                </div>
            </header>
            <slot />
        </div>
    </div>
</template>
  
<script setup lang="ts">
    
</script>