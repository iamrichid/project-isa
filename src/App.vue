<template>
  <div class="min-h-screen flex flex-col bg-gray-50">

    <!-- Header with Navigation -->
    <header class="flex items-center justify-between p-4 bg-white  mb-4">
      <img src="./assets/logo.jpg" width="160" alt="Logo" />

      <nav class="flex space-x-4">
         <button
          @click="activeTab = 'dashboard'"
          :class="activeTab === 'dashboard' ? 'text-blue-600 font-bold underline' : 'text-gray-600 hover:text-blue-500'"
          class="px-3 py-1"
        >
          Dashboard
        </button>
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
        Current Stock
          
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

    <!-- Preview Table -->
    <div class="overflow-x-auto bg-white shadow rounded-lg">
      <table class="min-w-full border-collapse text-sm">
        <thead class="bg-gray-100">
          <tr>
            <th class="p-2 border">PLIST-CODE</th>
            <th class="p-2 border">Tile Type</th>
            <th class="p-2 border">Size</th>
            <th class="p-2 border">BOX QTY</th>
            <th class="p-2 border">SIZE FACTOR</th>
            <th class="p-2 border">Opening Stock</th>
          </tr>
        </thead>
        <tbody>
          <tr
            v-for="(item, index) in openingStockArray"
            :key="index"
            class="hover:bg-gray-50"
          >
            <td class="p-2 border">{{ item['PLIST-CODE'] }}</td>
            <td class="p-2 border">{{ item['TILE TYPE'] }}</td>
            <td class="p-2 border">{{ item['SIZE'] }}</td>
            <td class="p-2 border">{{ item['BOX QTY'] }}</td>
            <td class="p-2 border">{{ item['SIZE FACTOR'] }}</td>
            <td class="p-2 border">{{ item['Opening Stock'] }}</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</div>


      <!-- Current Stock Tab -->
      <div v-if="activeTab === 'current'">
       <Inventory />
      </div>

      <!-- Transactions Tab -->
      <div v-if="activeTab === 'transactions'">
        <transaction />
      </div>

      <!-- Transactions Tab -->
      <div v-if="activeTab === 'dashboard'">
        <Dashboard :rawInventory="rawInventory"/>
      </div>

    </main>
  </div>
</template>

<script>
import Papa from 'papaparse';
import Inventory from './component/Inventory.vue';
import transaction from './component/transaction.vue';
import Dashboard from './component/dashboard.vue';

export default {
  components: {
    Inventory,
    transaction,
    Dashboard,
  },
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
        const cleaned = results.data.map(row => {
          const code = row['PLIST-CODE']?.trim();
          const boxQty = parseFloat(row['BOX QTY']) || 0;
          const sizeFactor = parseFloat(row['SIZE FACTOR']) || 1;
          const openingStock = +(boxQty * sizeFactor).toFixed(2); // rounded to 2dp

          if (code) {
            map[code] = openingStock;
            row['Opening Stock'] = openingStock;
          }

          return row;
        });

        this.openingStockMap = map;
        this.openingStockArray = cleaned;
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
