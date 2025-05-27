<template>
  <div>
    <h2 class="text-xl font-semibold mb-4">Upload Transaction CSV</h2>
    <input type="file" @change="handleCSV" accept=".csv" class="mb-4" />

    <div v-if="transactions.length">
      <h3 class="text-lg font-bold mb-2">Transaction Records</h3>
      <div class="flex justify-end mb-4">
  <button @click="showModal = true" class="bg-green-600 hover:bg-green-700 text-white px-4 py-2 rounded">
    + New Transaction
  </button>
</div>

      <div class="overflow-x-auto bg-white shadow rounded-lg">
        <table class="min-w-full border-collapse text-sm">
          <thead class="bg-gray-100 text-left">
            <tr>
              <th v-for="header in headers" :key="header" class="p-2 border">{{ header }}</th>
            </tr>
          </thead>
          <tbody>
            <tr
              v-for="(txn, index) in transactions"
              :key="index"
              class="hover:bg-gray-50"
            >
              <td class="p-2 border">{{ txn['WAY BILL NUMBER'] }}</td>
              <td class="p-2 border">{{ txn['CODE'] }}</td>
              <td class="p-2 border">{{ txn['QTY-M2'] }}</td>
              <td class="p-2 border">{{ txn['DATEOFISSUE'] }}</td>
              <td class="p-2 border">{{ txn['SIZE'] }}</td>
              <td class="p-2 border">{{ txn['CHX SIZE'] }}</td>
              <td class="p-2 border">{{ txn['QTY-PCS'] }}</td>
              <td class="p-2 border">{{ txn['DESTINATION'] }}</td>
              <td class="p-2 border">{{ txn['SALETYPE'] }}</td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
  </div>

  <!-- Add Transaction Modal -->
<div v-if="showModal" class="fixed inset-0 bg-black bg-opacity-50 flex items-center justify-center z-50">
  <div class="bg-white rounded-lg p-6 w-full max-w-2xl relative shadow-lg">
    <button class="absolute top-2 right-2 text-gray-500 hover:text-gray-800" @click="showModal = false">×</button>
    <h3 class="text-xl font-bold mb-4">Add New Transaction</h3>
    <form @submit.prevent="addTransaction" class="grid grid-cols-1 md:grid-cols-3 gap-4">
      <input v-model="newTransaction['WAY BILL NUMBER']" placeholder="WAY BILL NUMBER" class="p-2 border rounded" />
      <input v-model="newTransaction['CODE']" placeholder="CODE" class="p-2 border rounded" />
      <input v-model="newTransaction['QTY-M2']" type="number" placeholder="QTY-M2" class="p-2 border rounded" />
      <input v-model="newTransaction['DATEOFISSUE']" type="date" placeholder="Date of Issue" class="p-2 border rounded" />
      <input v-model="newTransaction['SIZE']" placeholder="SIZE" class="p-2 border rounded" />
      <input v-model="newTransaction['CHX SIZE']" placeholder="CHX SIZE" class="p-2 border rounded" />
      <input v-model="newTransaction['QTY-PCS']" type="number" placeholder="QTY-PCS" class="p-2 border rounded" />
      <input v-model="newTransaction['DESTINATION']" placeholder="DESTINATION" class="p-2 border rounded" />
      <input v-model="newTransaction['SALETYPE']" placeholder="SALE TYPE" class="p-2 border rounded" />

      <button type="submit" class="col-span-full bg-blue-600 hover:bg-blue-700 text-white px-4 py-2 rounded transition">
        Add Transaction
      </button>
    </form>
  </div>
</div>


</template>

<script setup>
import { ref, onMounted, watch } from 'vue'
import Papa from 'papaparse'

const transactions = ref([])
const showModal = ref(false)
const headers = [
  'WAY BILL NUMBER',
  'CODE',
  'QTY-M2',
  'DATEOFISSUE',
  'SIZE',
  'CHX SIZE',
  'QTY-PCS',
  'DESTINATION',
  'SALETYPE'
]

const newTransaction = ref({
  'WAY BILL NUMBER': '',
  'CODE': '',
  'QTY-M2': '',
  'DATEOFISSUE': '',
  'SIZE': '',
  'CHX SIZE': '',
  'QTY-PCS': '',
  'DESTINATION': '',
  'SALETYPE': ''
})

function handleCSV(event) {
  const file = event.target.files[0]
  if (!file) return

  Papa.parse(file, {
    header: true,
    skipEmptyLines: true,
    complete(results) {
      transactions.value = results.data
    }
  })

  onMounted(() => {
  const saved = localStorage.getItem('transactions')
  if (saved) transactions.value = JSON.parse(saved)
})

// Watch and save to localStorage
watch(transactions, (val) => {
  localStorage.setItem('transactions', JSON.stringify(val))
}, { deep: true })

function addTransaction() {
  if (!newTransaction.value['WAY BILL NUMBER'] || !newTransaction.value['CODE']) {
    alert('WAY BILL NUMBER and CODE are required')
    return
  }

  transactions.value.push({ ...newTransaction.value })

  for (let key in newTransaction.value) {
    newTransaction.value[key] = ''
  }

  showModal.value = false
}
}
</script>
