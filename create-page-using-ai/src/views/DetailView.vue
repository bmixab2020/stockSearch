<template>
  <div class="detail">
    <h1>Stock Ticker Search</h1>
    <input type="text" v-model="ticker" placeholder="Enter stock ticker">
    <button @click="search" :disabled="loading">{{ loading ? 'Searching...' : 'Search' }}</button>

    <div v-if="stock" class="stock-info">
      <h2>{{ stock.name }} ({{ stock.ticker }})</h2>
      <p><strong>Market Cap:</strong> {{ stock.marketCap }}</p>
      <p><strong>Volume:</strong> {{ stock.volume }}</p>
      <p><strong>Last Price:</strong> {{ stock.lastPrice }}</p>
    </div>
  </div>
</template>

<script>

export default {
  name: 'DetailView',
  data() {
    return {
      ticker: '',
      stock: null,
      loading: false
    }
  },
  methods: {
    async search() {
      if (!this.ticker) {
        return;
      }

      this.loading = true;
      this.stock = null;


      // Simulate API call
      setTimeout(() => {
        this.stock = {
          ticker: this.ticker.toUpperCase(),
          name:'Test',
          marketCap: '2.7T',
          volume: '54.1M',
          lastPrice: '172.26'
        };
        this.loading = false;
      }, 1000);
    }
  }
}
</script>

<style scoped>
.detail {
  max-width: 400px;
  margin: 0 auto;
  text-align: left;
}

.stock-info {
  margin-top: 20px;
  padding: 20px;
  border: 1px solid #ccc;
  border-radius: 5px;
}
</style>