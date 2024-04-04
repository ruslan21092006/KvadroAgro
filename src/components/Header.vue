<script setup>
defineProps({
  totalPrice: Number
})

const emit = defineEmits(['openDrawer'])
</script>

<template>
  <header class="flex flex-col items-center justify-between border-b border-slate-200 px-10 py-8">
    <router-link to="/">
      <div class="flex items-center flex-col md:flex-row gap-4 md:gap-0 md:items-center">
        <img src="/logo.png" alt="Logo" class="w-30" />
        <div>
          <h2 class="text-xl font-bold text-blue-600 uppercase">Kvadro Agro</h2>
          <p class="text-yellow-400">Магазин агротоварів</p>
        </div>
      </div>
    </router-link>

    <div class="flex flex-col items-center md:flex-row gap-10 mt-8 md:mt-0">
      <div class="flex items-center gap-3 text-gray-500 hover:text-black">
        <li @click="() => emit('openDrawer')"><img src="/cart.svg" alt="Cart" /></li>
        
      </div>

      <router-link to="/favorites">
        <div class="flex items-center gap-3 text-gray-500 hover:text-black">
          <img src="/heart.svg" alt="Cart" />
          <span>Закладки</span>
        </div>
      </router-link>

      <div class="flex items-center gap-3 text-gray-500 hover:text-black">
        <img src="/profile.svg" alt="Cart" />
        <span>Зв'язатися з продавцем <br>+380969380421<br></span>
      </div>
    </div>
  </header>
</template>