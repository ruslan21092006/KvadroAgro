<script setup>
import { reactive, watch, ref, onMounted } from 'vue'
import axios from 'axios'
import debounce from 'lodash.debounce'
import { inject } from 'vue'
import CardList from '../components/CardList.vue'

const { cart, addToCart, removeFromCart } = inject('cart')

const items = ref([])

const filters = reactive({
  selectedType: ref('species'), // Вибраний тип
  searchQuery: ref('')
})

const onClickAddPlus = (item) => {
  if (!item.isAdded) {
    addToCart(item)
  } else {
    removeFromCart(item)
  }
}

const onChangeSelect = (event) => {
  filters.selectedType = event.target.value;
  fetchItems();
};

const onChangeSearchInput = debounce((event) => {
  filters.searchQuery = event.target.value
}, 300)

const addToFavorite = async (item) => {
  try {
    if (!item.isFavorite) {
      const obj = {
        item_id: item.id
      }

      item.isFavorite = true

      const { data } = await axios.post(`https://65faed1e3909a9a65b1c07f4.mockapi.io/favorites`, obj)

      item.favoriteId = data.id
    } else {
      item.isFavorite = false
      await axios.delete(`https://65faed1e3909a9a65b1c07f4.mockapi.io/favorites/${item.favoriteId}`)
      item.favoriteId = null
    }
  } catch (err) {
    console.log(err)
  }
}

const fetchFavorites = async () => {
  try {
    const { data: favorites } = await axios.get(`https://65faed1e3909a9a65b1c07f4.mockapi.io/favorites`)

    items.value = items.value.map((item) => {
      const favorite = favorites.find((favorite) => favorite.item_id === item.id)

      if (!favorite) {
        return item
      }

      return {
        ...item,
        isFavorite: true,
        favoriteId: favorite.id
      }
    })
  } catch (err) {
    console.log(err)
  }
}

const fetchItems = async () => {
  try {
    const params = {
      species: filters.selectedType,
      title: filters.searchQuery
    };

    if (filters.searchQuery) {
      params.title = `${filters.searchQuery}`
    }

    const { data } = await axios.get(`https://65faed1e3909a9a65b1c07f4.mockapi.io/item`, {
      params
    })  



    // items.value = data.filter(item => item.type === filters.selectedType)
    items.value = data.map((obj) => ({
      ...obj,
      isFavorite: false,
      favoriteId: null,
      isAdded: false
    }))
    
  } catch (err) {
    console.log(err)
  }
}

onMounted(async () => {
  const localCart = localStorage.getItem('cart')
  cart.value = localCart ? JSON.parse(localCart) : []

  await fetchItems()
  await fetchFavorites()

  items.value = items.value.map((item) => ({
    ...item,
    isAdded: cart.value.some((cartItem) => cartItem.id === item.id)
  }))
})

watch(cart, () => {
  items.value = items.value.map((item) => ({
    ...item,
    isAdded: false
  }))
})

watch(filters, fetchItems)
</script>

<template>
  <div class="flex flex-col lg:flex-row justify-between items-center mb-8">
    <h2 class="text-3xl text-blue-400 font-bold">Каталог товарів</h2>

    <div class="flex flex-col lg:flex-row lg:gap-4 mt-4 lg:mt-0">
      <select @change="onChangeSelect" class="py-2 px-3  bg-yellow-50 border border-blue-300 rounded-md outline-none">
        <option>Виберіть вид товару</option>
        <option value="zerno">Посівний матеріал</option>
        <option value="zzr">Засоби захисту рослин</option>
        <option value="dobr">Добрива</option>
      </select>

      <div class="relative border border-blue-300 rounded-md lg:flex-row lg:gap-4">
        <img class="absolute left-20 top-3" src="/search.svg" />
        <input
          @input="onChangeSearchInput"
          class="py-2 px-3 border rounded-md outline-none  bg-yellow-50 placeholder:text-sm placeholder:text-gray-500"
          type="text"
          placeholder="Пошук..."
        />
      </div>
    </div>
  </div>

  <div class="mt-10">
    <CardList :items="items" @add-to-favorite="addToFavorite" @add-to-cart="onClickAddPlus" />
  </div>
</template>