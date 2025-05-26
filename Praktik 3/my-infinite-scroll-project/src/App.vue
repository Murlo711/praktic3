<template>
  <div>
    <h1>Список товаров</h1>
    <div class="product-list">
      <div class="product" v-for="product in products" :key="product.id">
        <img
          v-if="product.images && product.images.length > 0"
          :src="product.images[0]"
          :alt="`Фото ${product.title}`"
          class="product-image"
        />
        <h3>{{ product.title }}</h3>
        <p>{{ product.description }}</p>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, onMounted } from 'vue';

export default {
  setup() {
    const products = ref([]);
    const totalProducts = 190;
    const limit = 12;
    const skip = ref(0);
    const loadedCount = ref(0);
    const loading = ref(false);

    const loadProducts = async () => {
      if (loading.value || loadedCount.value >= totalProducts) return;

      loading.value = true;

      const remaining = totalProducts - loadedCount.value;
      const currentLimit = remaining >= limit ? limit : remaining;

      try {
        const response = await fetch(`https://dummyjson.com/products?limit=${currentLimit}&skip=${skip.value}`);
        const data = await response.json();

        products.value.push(...data.products);
        loadedCount.value += data.products.length;
        skip.value += data.products.length;

        if (loadedCount.value >= totalProducts) {
          window.removeEventListener('scroll', handleScroll);
        }
      } catch (error) {
        console.error('Ошибка при загрузке данных:', error);
      } finally {
        loading.value = false;
      }
    };

    const handleScroll = () => {
      const scrollPosition = window.innerHeight + window.scrollY;
      const threshold = document.body.offsetHeight - 100;

      if (scrollPosition >= threshold) {
        loadProducts();
      }
    };

    onMounted(() => {
      loadProducts();
      window.addEventListener('scroll', handleScroll);
    });

    return { products };
  }
};
</script>

<style>
body {
  font-family: Arial, sans-serif;
  margin: 0;
  padding: 20px;
}
.product-list {
  display: flex;
  flex-wrap: wrap;
  gap: 16px;
}
.product {
  width: calc(25% - 16px);
  border: 1px solid #ccc;
  padding: 8px;
  box-sizing: border-box;
}
.product-image {
  width: 100%;
  height: auto;
  margin-bottom: 8px;
}
</style>