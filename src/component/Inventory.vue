<template>
  <div>
    <!-- Upload -->
    <div class="mb-4">
      <input type="file" accept=".csv" @change="handleCSVUpload" class="border p-2 rounded" />
    </div>

    <!-- Search -->
   <!-- Filters Section -->
<div class="flex flex-wrap gap-4 items-end mb-6 bg-white p-4 rounded-xl shadow-md">
  <!-- Search -->
  <div class="w-full md:w-1/3">
    <label class="block text-sm font-medium text-gray-700 mb-1">Search</label>
    <input
      v-model="searchQuery"
      type="text"
      placeholder="Search Tile Code or Type..."
      class="w-full border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
    />
  </div>

  <!-- Size Filter -->
  <div class="w-full md:w-1/5">
    <label class="block text-sm font-medium text-gray-700 mb-1">Tile Size</label>
    <select
      v-model="selectedSize"
      class="w-full border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
    >
      <option value="">All Sizes</option>
      <option v-for="size in uniqueSizes" :key="size" :value="size">{{ size }}</option>
    </select>
  </div>

  <!-- Type Filter -->
  <div class="w-full md:w-1/5">
    <label class="block text-sm font-medium text-gray-700 mb-1">Tile Type</label>
    <select
      v-model="selectedType"
      class="w-full border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
    >
      <option value="">All Types</option>
      <option v-for="type in uniqueTypes" :key="type" :value="type">{{ type }}</option>
    </select>
  </div>

  <!-- Min Stock -->
  <div class="w-1/2 md:w-1/6">
    <label class="block text-sm font-medium text-gray-700 mb-1">Min Stock</label>
    <input
      type="number"
      v-model.number="stockMin"
      placeholder="0"
      class="w-full border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
    />
  </div>

  <!-- Max Stock -->
  <div class="w-1/2 md:w-1/6">
    <label class="block text-sm font-medium text-gray-700 mb-1">Max Stock</label>
    <input
      type="number"
      v-model.number="stockMax"
      placeholder="1000"
      class="w-full border border-gray-300 rounded-lg p-2 focus:outline-none focus:ring-2 focus:ring-blue-500"
    />
  </div>
</div>


    <!-- Table -->
    <div class="overflow-auto">
      <div class="overflow-x-auto rounded-2xl shadow-lg bg-white">
  <table class="min-w-full text-sm text-gray-800">
    <thead class="bg-gradient-to-r from-gray-100 to-gray-200 text-gray-600 uppercase tracking-wider">
      <tr>
        <th
          v-for="header in headers"
          :key="header"
          class="px-4 py-3 text-left whitespace-nowrap"
        >
          {{ header }}
        </th>
      </tr>
    </thead>
    <tbody>
      <tr
        v-for="(item, index) in filteredInventory"
        :key="index"
        class="hover:bg-gray-50 transition duration-200 ease-in-out border-b border-gray-200"
      >
        <td class="px-4 py-3 font-medium text-gray-900">{{ item['Tile Code'] }}</td>
        <td class="px-4 py-3 font-bold text-green-600">{{ calculateCurrentStock(item) }}</td>
        <td class="px-4 py-3">{{ item['Opening Stock'] }}</td>
        <td class="px-4 py-3">{{ item['Stock In'] }}</td>
        <td class="px-4 py-3">{{ item['Stock Out'] }}</td>
        <td class="px-4 py-3">{{ item['SAMPLEOUT'] }}</td>
        <td class="px-4 py-3">{{ item['SOLDOUT'] }}</td>
        <td class="px-4 py-3">{{ item['Adjustments'] }}</td>
        <td class="px-4 py-3">{{ item['TILESIZE'] }}</td>
        <td class="px-4 py-3">{{ item['TILETYPE'] }}</td>
        <td class=" text-center p-2 relative">
  <button @click="toggleDropdown(index)" class="p-1 rounded hover:bg-gray-200">
    ⋮
  </button>

  <div
    v-if="activeDropdown === index"
    class="absolute right-0 mt-2 w-32 bg-white border rounded shadow-lg z-50"
  >
    <button class="block w-full text-left px-4 py-2 hover:bg-gray-100" @click="viewItem(item)">View</button>
    <button class="block w-full text-left px-4 py-2 hover:bg-gray-100" @click="editItem(item)">Edit</button>
    <button class="block w-full text-left px-4 py-2 text-red-600 hover:bg-gray-100" @click="deleteItem(index)">Delete</button>
  </div>
</td>
        </tr>
    </tbody>
  </table>
</div>

    </div>
  </div>



  <!-- View Modal -->
<div v-if="showViewModal" class="fixed inset-0 bg-black bg-opacity-30 flex items-center justify-center z-50">
  <div class="bg-white p-6 rounded shadow-lg w-96">
    <h3 class="text-lg font-bold mb-2">View Item</h3>
    <pre class="text-sm">{{ editedItem }}</pre>
    <button class="mt-4 bg-gray-700 text-white px-4 py-1 rounded" @click="showViewModal = false">Close</button>
  </div>
</div>

<!-- Edit Modal -->
<!-- Edit Modal -->
<div
  v-if="showEditModal"
  class="fixed inset-0 bg-black bg-opacity-30 flex items-center justify-center z-50"
>
  <div class="bg-white p-6 rounded shadow-lg w-full max-w-lg max-h-[90vh] overflow-auto">
    <h3 class="text-lg font-bold mb-4">Edit Item</h3>

    <div v-for="(value, key) in editedItem" :key="key" class="mb-3">
      <label class="block text-sm font-semibold mb-1 capitalize">{{ key }}</label>
      <input
        v-model="editedItem[key]"
        class="border rounded w-full p-2 text-sm"
        :placeholder="key"
      />
    </div>

    <div class="flex gap-2 justify-end mt-6">
      <button class="bg-green-600 text-white px-4 py-2 rounded" @click="saveEdit">Save</button>
      <button class="bg-gray-500 text-white px-4 py-2 rounded" @click="showEditModal = false">Cancel</button>
    </div>
  </div>
</div>
</template>

<script>
import * as Papa from 'papaparse'

export default {
  data() {
    return {
      rawInventory: [],
      searchQuery: '',
      headers: [
        'Tile Code',
        'Current Stock',
        'Opening Stock',
        'Stock In',
        'Stock Out',
        'SAMPLEOUT',
        'SOLDOUT',
        'Adjustments',
        'TILESIZE',
        'TILETYPE',
        'ACTIONS',
      ],
       searchQuery: '',
        selectedSize: '',
        selectedType: '',
        stockMin: null,
        stockMax: null,
        sortAsc: true,
        editedIndex: null,
        editedItem: null,
        showEditModal: false,
        showViewModal: false,
         activeDropdown: null, 
    }
  },
  computed: {
  uniqueSizes() {
    const sizes = this.rawInventory.map(item => item.TILESIZE).filter(Boolean)
    return [...new Set(sizes)]
  },
  uniqueTypes() {
    const types = this.rawInventory.map(item => item.TILETYPE).filter(Boolean)
    return [...new Set(types)]
  },
  filteredInventory() {
    const q = this.searchQuery.toLowerCase()

    return this.rawInventory
      .filter(item => {
        const matchQuery =
          item['Tile Code']?.toLowerCase().includes(q) ||
          item['TILETYPE']?.toLowerCase().includes(q)

        const matchSize = this.selectedSize ? item.TILESIZE === this.selectedSize : true
        const matchType = this.selectedType ? item.TILETYPE === this.selectedType : true

        const currentStock = parseFloat(this.calculateCurrentStock(item))

        const matchStockMin = this.stockMin != null ? currentStock >= this.stockMin : true
        const matchStockMax = this.stockMax != null ? currentStock <= this.stockMax : true

        return matchQuery && matchSize && matchType && matchStockMin && matchStockMax
      })
      .sort((a, b) => {
        const codeA = a['Tile Code'] || ''
        const codeB = b['Tile Code'] || ''
        return this.sortAsc ? codeA.localeCompare(codeB) : codeB.localeCompare(codeA)
      })
  },
},

  methods: {
    handleCSVUpload(event) {
      const file = event.target.files[0]
      if (!file) return

      Papa.parse(file, {
        header: true,
        skipEmptyLines: true,
        complete: (result) => {
          this.rawInventory = result.data
          localStorage.setItem('inventory_data', JSON.stringify(result.data))
        },
      })
    },

    toggleSort() {
  this.sortAsc = !this.sortAsc
},

  toggleDropdown(index) {
    this.activeDropdown = this.activeDropdown === index ? null : index;
  },
  viewItem(item) {
    this.selectedItem = item;
    this.showViewModal = true;
    this.activeDropdown = null;
  },
  editItem(item) {
    this.editedItem = { ...this.rawInventory[item] };
    this.showEditModal = true;
    this.activeDropdown = null;
  },
  deleteItem(index) {
    if (confirm("Are you sure you want to delete this item?")) {
      this.rawInventory.splice(index, 1);
    }
    // Optional: trigger a toast or modal confirm
  },
  saveEdit() {
     if (this.editedIndex !== null) {
      this.$set(this.rawInventory, this.editedIndex, this.editedItem);
      this.showEditModal = false;
    }
  },

    calculateCurrentStock(row) {
      const parse = (val) => parseFloat(val) || 0

      return (
        parse(row['Opening Stock']) +
        parse(row['Stock In']) -
        parse(row['Stock Out']) -
        parse(row['SAMPLEOUT']) -
        parse(row['SOLDOUT']) +
        parse(row['Adjustments'])
      ).toFixed(2)
    },
  },
  mounted() {
    const saved = localStorage.getItem('inventory_data')
    if (saved) {
      this.rawInventory = JSON.parse(saved)
    }
  },
}
</script>
