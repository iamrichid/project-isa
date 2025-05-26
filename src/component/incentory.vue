<template>
  <div>
    <!-- Upload -->
    <div class="mb-4">
      <input type="file" accept=".csv" @change="handleCSVUpload" class="border p-2 rounded" />
    </div>

    <!-- Search -->
    <div class="flex flex-wrap gap-4 items-center mb-4">
      <input
        v-model="searchQuery"
        type="text"
        placeholder="Search by Tile Code or Type..."
        class="border p-2 rounded w-full md:w-1/3"
      />
    </div>
    <!-- Filters -->
<div class="flex flex-wrap gap-4 items-center mb-4">
  <input
    v-model="searchQuery"
    type="text"
    placeholder="Search Tile Code or Type"
    class="border p-2 rounded w-full md:w-1/4"
  />

  <select v-model="selectedSize" class="border p-2 rounded w-full md:w-1/5">
    <option value="">All Sizes</option>
    <option v-for="size in uniqueSizes" :key="size" :value="size">{{ size }}</option>
  </select>

  <select v-model="selectedType" class="border p-2 rounded w-full md:w-1/5">
    <option value="">All Types</option>
    <option v-for="type in uniqueTypes" :key="type" :value="type">{{ type }}</option>
  </select>

  <input
    type="number"
    v-model.number="stockMin"
    placeholder="Min Stock"
    class="border p-2 rounded w-1/6"
  />

  <input
    type="number"
    v-model.number="stockMax"
    placeholder="Max Stock"
    class="border p-2 rounded w-1/6"
  />
</div>

    <!-- Table -->
    <div class="overflow-auto">
      <table class="min-w-full border border-gray-300">
        <thead class="bg-gray-200">
          <tr>
            <th v-for="header in headers" :key="header" class="border p-2 text-left">
              {{ header }}
            </th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(item, index) in filteredInventory" :key="index" class="hover:bg-gray-100">
            
            <td class="border p-2">{{ item['Tile Code'] }}</td>
            <td class="border p-2 font-bold">{{ calculateCurrentStock(item) }}</td>
            <td class="border p-2">{{ item['Opening Stock'] }}</td>
            <td class="border p-2">{{ item['Stock In'] }}</td>
            <td class="border p-2">{{ item['Stock Out'] }}</td>
            <td class="border p-2">{{ item['SAMPLEOUT'] }}</td>
            <td class="border p-2">{{ item['SOLDOUT'] }}</td>
            <td class="border p-2">{{ item['Adjustments'] }}</td>
            <td class="border p-2">{{ item['TILESIZE'] }}</td>
            <td class="border p-2">{{ item['TILETYPE'] }}</td>
          </tr>
        </tbody>
      </table>
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
      ],
       searchQuery: '',
        selectedSize: '',
        selectedType: '',
        stockMin: null,
        stockMax: null,
        sortAsc: true,
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
