<script setup>
import { ref, computed } from 'vue'

const description = ref('')
const amount = ref('')
const type = ref('expense')
const category = ref('Food')

const transactions = ref([])

function addTransaction() {
  if (description.value === '' || amount.value === '') {
    alert('Please fill all fields')
    return
  }

  transactions.value.push({
    description: description.value,
    amount: Number(amount.value),
    type: type.value,
    category: category.value
  })

  description.value = ''
  amount.value = ''
}
function deleteTransaction(index) {
  transactions.value.splice(index, 1)
}

const totalIncome = computed(() => {
  return transactions.value
    .filter(transaction => transaction.type === 'income')
    .reduce((total, transaction) => total + transaction.amount, 0)
})

const totalExpenses = computed(() => {
  return transactions.value
    .filter(transaction => transaction.type === 'expense')
    .reduce((total, transaction) => total + transaction.amount, 0)
})

const balance = computed(() => {
  return totalIncome.value - totalExpenses.value
})
const expensesByCategory = computed(() => {
  const categories = {}

  transactions.value
    .filter(transaction => transaction.type === 'expense')
    .forEach(transaction => {
      if (!categories[transaction.category]) {
        categories[transaction.category] = 0
      }

      categories[transaction.category] += transaction.amount
    })

  return categories
})

const maxExpense = computed(() => {
  const values = Object.values(expensesByCategory.value)

  if (values.length === 0) {
    return 0
  }

  return Math.max(...values)
})
</script>

<template>
  <div class="app">
    <h1>💰 Expense Tracker</h1>

    <div class="summary">
      <div class="card">
        <h3>Balance</h3>
        <p>${{ balance }}</p>
      </div>

      <div class="card">
        <h3>Income</h3>
        <p>${{ totalIncome }}</p>
      </div>

      <div class="card">
        <h3>Expenses</h3>
        <p>${{ totalExpenses }}</p>
      </div>
    </div>

    <div class="form">
      <h2>Add Transaction</h2>

      <input v-model="description" placeholder="Description" />
      <input v-model="amount" type="number" placeholder="Amount" />

      <select v-model="type">
        <option value="expense">Expense</option>
        <option value="income">Income</option>
      </select>

      <select v-model="category">
        <option value="Food">Food</option>
        <option value="Transport">Transport</option>
        <option value="Shopping">Shopping</option>
        <option value="Bills">Bills</option>
        <option value="Other">Other</option>
      </select>

      <button @click="addTransaction">Add Transaction</button>
    </div>

    <div class="chart">
      <h2>Expenses by Category</h2>

      <p v-if="Object.keys(expensesByCategory).length === 0">No expenses yet.</p>

      <div
        v-for="(amount, category) in expensesByCategory"
        :key="category"
        class="chart-row"
      >
        <div class="category-name">{{ category }}</div>

        <div class="bar-container">
          <div
            class="bar"
            :style="{
              width: (amount / maxExpense) * 100 + '%'
            }"
          >
            ${{ amount }}
          </div>
        </div>
      </div>
    </div>

    <div class="transactions">
      <h2>Transactions</h2>

      <p v-if="transactions.length === 0">No transactions yet.</p>

      <div
        v-for="(transaction, index) in transactions"
        :key="index"
        class="transaction"
      >
        <div>
          <strong>{{ transaction.description }}</strong>
          <small>{{ transaction.category }}</small>
        </div>

        <div class="transaction-right">
          <span>
            {{ transaction.type === 'income' ? '+' : '-' }}
            ${{ transaction.amount }}
          </span>

          <button class="delete-button" @click="deleteTransaction(index)">🗑️</button>
        </div>
      </div>
    </div>
  </div>
</template>
