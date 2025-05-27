<template>
  <section class="p-6 bg-white rounded-lg shadow space-y-6 max-w-7xl mx-auto">
    <h1 class="text-3xl font-bold mb-6 text-gray-800">Current Stock Dashboard</h1>

    <div class="grid grid-cols-1 md:grid-cols-4 gap-6">
      <!-- Total Stock -->
      <div class="bg-blue-600 text-white rounded-lg p-6 shadow hover:shadow-lg transition">
        <h2 class="text-lg font-semibold mb-2">Total Stock</h2>
        <p class="text-4xl font-extrabold">{{ totalCurrentStock }}</p>
      </div>

      <!-- Low Stock -->
      <div class="bg-red-600 text-white rounded-lg p-6 shadow hover:shadow-lg transition">
        <h2 class="text-lg font-semibold mb-2">Low Stock Items</h2>
        <p class="text-4xl font-extrabold">{{ lowStockItems }}</p>
      </div>

      <!-- Top Tile Types -->
      <div class="bg-green-600 text-white rounded-lg p-6 shadow hover:shadow-lg transition md:col-span-2">
        <h2 class="text-lg font-semibold mb-4">Top 5 Tile Types</h2>
        <ul>
          <li
            v-for="item in topTileTypes"
            :key="item.type"
            class="flex justify-between py-2 border-b border-green-300"
          >
            <span>{{ item.type }}</span>
            <span class="font-extrabold">{{ item.stock }}</span>
          </li>
        </ul>
      </div>
    </div>

    <!-- Stock by Size -->
    <div>
      <h2 class="text-lg font-semibold mb-4 text-gray-700">Stock by Size</h2>
      <div class="grid grid-cols-2 md:grid-cols-5 gap-4">
        <div
          v-for="item in stockBySize"
          :key="item.size"
          class="bg-gray-100 rounded-lg p-4 text-center shadow hover:shadow-md transition"
        >
          <p class="font-semibold text-gray-900">{{ item.size }}</p>
          <p class="text-3xl font-extrabold text-gray-700">{{ item.stock }}</p>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
export default {
  props: ['rawInventory'],
  computed: {
    totalCurrentStock() {
      return this.rawInventory.reduce(
        (sum, item) => sum + Number(item['Current Stock'] || 0),
        0
      )
    },
    lowStockItems() {
      return this.rawInventory.filter(
        (item) => Number(item['Current Stock'] || 0) < 10
      ).length
    },
    topTileTypes() {
      const typeMap = {}
      this.rawInventory.forEach((item) => {
        const type = item['TILETYPE'] || 'Unknown'
        const stock = Number(item['Current Stock'] || 0)
        if (!typeMap[type]) typeMap[type] = 0
        typeMap[type] += stock
      })
      return Object.entries(typeMap)
        .sort((a, b) => b[1] - a[1])
        .slice(0, 5)
        .map(([type, stock]) => ({ type, stock }))
    },
    stockBySize() {
      const sizeMap = {}
      this.rawInventory.forEach((item) => {
        const size = item['TILESIZE'] || 'Unknown'
        const stock = Number(item['Current Stock'] || 0)
        if (!sizeMap[size]) sizeMap[size] = 0
        sizeMap[size] += stock
      })
      return Object.entries(sizeMap).map(([size, stock]) => ({ size, stock }))
    }
  }
}
</script>

<style scoped>
/* Optional: add some subtle hover or transitions if you want */
</style>
