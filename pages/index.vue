<template>
    <div>

        <div class="flex justify-center mb-[40px] flex-col items-center text-center sm:flex-row sm:text-left gap-[20px]">
            <figure class="relative bg-red-500 w-[250px] h-[300px] overflow-hidden rounded-[20px] min-w-[250px]">
                <img class="profile-img" src="/images/profile2.jpg" />
            </figure>
            <h2 class="max-w-[300px] text-4xl/13 flex justify-center flex-col">
                <span class="block">Hi, I'm <b>Neil</b>,</span> An Experienced Web Developer
            </h2>
        </div>

        <div>
            <h2 class="mb-[24px]">
                Projects
            </h2>
            <div class="flex justify-between gap-[8px] sm:flex-nowrap flex-wrap">
                <CommonCard />
                <CommonCard />
                <CommonCard />
            </div>
        </div>
    </div>
</template>

<script setup lang="ts">
    const appConfig = useAppConfig()
</script>

<style lang="css" scoped>
    .profile-img {
        height: auto;
        width: 319px;
        position: absolute;
        left: 50%;
        transform: translateX(-50%);
        max-width: 1080px;
        filter: saturate(0.9) brightness(0.9);
    }
</style>