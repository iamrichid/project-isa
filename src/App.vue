<template>
  <div class="min-h-screen flex flex-col bg-gray-50">

    <!-- Header with Navigation -->
    <header class="flex items-center justify-between p-4 bg-white  mb-4">
      <img src="./assets/logo.jpg" width="160" alt="Logo" />

      <nav class="flex space-x-4">
        <button
          @click="activeTab = 'opening'"
          :class="activeTab === 'opening' ? 'text-blue-600 font-bold underline' : 'text-gray-600 hover:text-blue-500'"
          class="px-3 py-1"
        >
          Opening Stock
        </button>
        <button
          @click="activeTab = 'current'"
          :class="activeTab === 'current' ? 'text-blue-600 font-bold underline' : 'text-gray-600 hover:text-blue-500'"
          class="px-3 py-1"
        >
          
        </button>
        <button
          @click="activeTab = 'transactions'"
          :class="activeTab === 'transactions' ? 'text-blue-600 font-bold underline' : 'text-gray-600 hover:text-blue-500'"
          class="px-3 py-1"
        >
          Transactions
        </button>
      </nav>
    </header>

    <!-- Main Content Area -->
    <main class="flex-grow container mx-auto p-4">

      <!-- Opening Stock Tab -->
      <div v-if="activeTab === 'opening'">
        <h2 class="text-xl font-semibold mb-4">Upload Opening Stock CSV</h2>
        <input type="file" @change="handleOpeningCSV" accept=".csv" class="mb-4" />

        <div v-if="Object.keys(openingStockMap).length">
          <p class="mb-2 text-green-600">Opening stock loaded successfully.</p>
        </div>
      </div>

      <!-- Current Stock Tab -->
      <div v-if="activeTab === 'current'">
        <h2 class="text-xl font-semibold mb-4">Upload Current Stock CSV</h2>
        <input type="file" @change="handleCurrentCSV" accept=".csv" class="mb-4" />

        <div v-if="rawInventory.length">
          <h3 class="text-lg font-bold mt-6 mb-2">Inventory Table</h3>
          <div class="overflow-x-auto bg-white shadow rounded-lg">
            <table class="min-w-full border-collapse">
              <thead class="bg-gray-100 text-left text-sm">
                <tr>
                  <th class="p-2 border">Tile Code</th>
                  <th class="p-2 border">Opening Stock</th>
                  <th class="p-2 border">Current Stock</th>
                  <th class="p-2 border">TILE SIZE</th>
                  <th class="p-2 border">TILE TYPE</th>
                  <!-- Add more columns as needed -->
                </tr>
              </thead>
              <tbody>
                <tr
                  v-for="(item, index) in rawInventory"
                  :key="index"
                  class="hover:bg-gray-50"
                >
                  <td class="p-2 border">{{ item['Tile Code'] }}</td>
                  <td class="p-2 border">{{ item['Opening Stock'] || '—' }}</td>
                  <td class="p-2 border">{{ item['Current Stock'] }}</td>
                  <td class="p-2 border">{{ item['TILESIZE'] }}</td>
                  <td class="p-2 border">{{ item['TILETYPE'] }}</td>
                </tr>
              </tbody>
            </table>
          </div>
        </div>
      </div>

      <!-- Transactions Tab -->
      <div v-if="activeTab === 'transactions'">
        <h2 class="text-xl font-semibold mb-4">Transactions</h2>
        <p class="text-gray-500">This section will handle stock in/out logs, sample outs, sold outs, and adjustments. (Coming soon!)</p>
      </div>

    </main>
  </div>
</template>

<script>
import Papa from 'papaparse';

export default {
  data() {
    return {
      activeTab: 'current',
      rawInventory: [],
      openingStockMap: {},
    };
  },
  methods: {
    handleCurrentCSV(event) {
      const file = event.target.files[0];
      if (!file) return;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          this.rawInventory = results.data;
          this.syncOpeningStock();
        },
      });
    },
    handleOpeningCSV(event) {
      const file = event.target.files[0];
      if (!file) return;

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (results) => {
          const map = {};
          results.data.forEach(row => {
            const code = row['PLIST-CODE']?.trim();
            const boxQty = parseFloat(row['BOX QTY']) || 0;
            const sizeFactor = parseFloat(row['SIZE FACTOR']) || 1;
            const openingStock = boxQty * sizeFactor;

            if (code) {
              map[code] = openingStock;
            }
          });

          this.openingStockMap = map;
          this.syncOpeningStock();
        },
      });
    },
    syncOpeningStock() {
      this.rawInventory = this.rawInventory.map(item => {
        const code = item['Tile Code']?.trim();
        if (code && this.openingStockMap[code] !== undefined) {
          item['Opening Stock'] = this.openingStockMap[code];
        }
        return item;
      });
    }
  }
};
</script>

<style scoped>
/* Optional style overrides */
</style>
