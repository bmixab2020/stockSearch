<template>
  <div class="expenses-tracker">
    <h1>Expense Tracker</h1>

    <div class="list-manager">
      <input type="text" v-model="newListName" placeholder="New list name" @keyup.enter="createList">
      <button @click="createList" class="btn btn-primary">Create List</button>
      <select v-model="activeList" @change="loadList">
        <option v-for="listName in Object.keys(lists)" :key="listName" :value="listName">{{ listName }}</option>
      </select>
      <button @click="deleteList" :disabled="!activeList" class="btn btn-danger">Delete List</button>
    </div>

    <div v-if="activeList">
      <div class="transaction-form">
        <select v-model="newTransaction.type">
          <option value="expense">Expense</option>
          <option value="income">Income</option>
        </select>
        <input type="text" v-model="newTransaction.description" placeholder="Description">
        <select v-model="newTransaction.category">
          <option v-for="category in categories" :key="category" :value="category">{{ category }}</option>
        </select>
        <input type="number" v-model="newTransaction.amount" placeholder="Amount">
        <button @click="addTransaction" class="btn btn-primary">Add Transaction</button>
      </div>

      <div class="filters">
        <input type="text" v-model="filterCategory" placeholder="Filter by category">
        <select v-model="sortBy">
          <option value="date-desc">Sort by Date (Newest First)</option>
          <option value="date-asc">Sort by Date (Oldest First)</option>
          <option value="amount-desc">Sort by Amount (High to Low)</option>
          <option value="amount-asc">Sort by Amount (Low to High)</option>
        </select>
        <button @click="resetData" class="btn btn-danger">Clear/Reset All Data</button>
        <button @click="printPage" class="btn btn-primary">Print</button>
      </div>

      <div class="printable">
        <h2>Transactions for {{ activeList }}</h2>
        <div class="transactions">
          <div class="income">
            <h3>Income</h3>
            <ul>
              <li v-for="transaction in filteredAndSortedIncome" :key="transaction.id">
                <span>{{ transaction.description }} ({{ transaction.category }})</span>
                <span>{{ formatCurrency(transaction.amount) }}</span>
              </li>
            </ul>
          </div>
          <div class="expenses-list">
            <h3>Expenses</h3>
            <ul>
              <li v-for="transaction in filteredAndSortedExpenses" :key="transaction.id">
                <span>{{ transaction.description }} ({{ transaction.category }})</span>
                <span>{{ formatCurrency(transaction.amount) }}</span>
              </li>
            </ul>
          </div>
        </div>

        <div class="summary">
          <h3>Summary Breakdown by Category (Expenses)</h3>
          <ul>
            <li v-for="(total, category) in categorySummary" :key="category">
              <span>{{ category }}</span>
              <span>{{ formatCurrency(total) }}</span>
            </li>
          </ul>
        </div>
      </div>
    </div>
    <div v-else>
      <p>Create a list to start tracking your expenses.</p>
    </div>
  </div>
</template>

<script>
export default {
  name: 'ExpensesView',
  data() {
    return {
      newTransaction: {
        type: 'expense',
        description: '',
        category: 'Food',
        amount: null
      },
      transactions: [],
      filterCategory: '',
      sortBy: 'date-desc',
      categories: ['Salary', 'Food', 'Entertaining', 'Utility', 'Rent', 'Groceries', 'Transportation', 'Bonus', 'Gift', 'Other'],
      lists: {},
      activeList: null,
      newListName: ''
    }
  },
  computed: {
    
    income() {
      return this.transactions.filter(t => t.type === 'income');
    },
    expenses() {
      return this.transactions.filter(t => t.type === 'expense');
    },
    filteredAndSortedTransactions() {
      let transactions = [...this.transactions];

      if (this.filterCategory) {
        transactions = transactions.filter(t => t.category.toLowerCase().includes(this.filterCategory.toLowerCase()));
      }

      transactions.sort((a, b) => {
        switch (this.sortBy) {
          case 'date-asc':
            return a.id - b.id;
          case 'amount-desc':
            return b.amount - a.amount;
          case 'amount-asc':
            return a.amount - b.amount;
          case 'date-desc':
          default:
            return b.id - a.id;
        }
      });

      return transactions;
    },
    filteredAndSortedIncome() {
        return this.filteredAndSortedTransactions.filter(t => t.type === 'income');
    },
    filteredAndSortedExpenses() {
        return this.filteredAndSortedTransactions.filter(t => t.type === 'expense');
    },
    categorySummary() {
      return this.expenses.reduce((summary, transaction) => {
        if (transaction.category) {
          if (!summary[transaction.category]) {
            summary[transaction.category] = 0;
          }
          summary[transaction.category] += transaction.amount;
        }
        return summary;
      }, {});
    }
  },
  methods: {
    addTransaction() {
      if (this.newTransaction.description && this.newTransaction.amount) {
        this.transactions.push({
          id: Date.now(),
          ...this.newTransaction
        });
        this.newTransaction.description = '';
        this.newTransaction.category = this.categories[0];
        this.newTransaction.amount = null;
      }
    },
    resetData() {
      this.transactions = [];
    },
    createList() {
      if (this.newListName && !this.lists[this.newListName]) {
        this.lists[this.newListName] = [];
        this.activeList = this.newListName;
        this.transactions = [];
        this.newListName = '';
      }
    },
    deleteList() {
      if (this.activeList && confirm('Are you sure you want to delete this list?')) {
        delete this.lists[this.activeList];
        const listKeys = Object.keys(this.lists);
        this.activeList = listKeys.length > 0 ? listKeys[0] : null;
        this.loadList();
      }
    },
    loadList() {
      if (this.activeList) {
        this.transactions = this.lists[this.activeList];
      } else {
        this.transactions = [];
      }
    },
    saveLists() {
      if (this.activeList) {
        this.lists[this.activeList] = this.transactions;
      }
      localStorage.setItem('expense-tracker-lists', JSON.stringify(this.lists));
    },
    printPage() {
      window.print();
    },
    formatCurrency(value) {
      if (typeof value !== 'number') {
        return value;
      }
      const formatter = new Intl.NumberFormat('en-US', {
        style: 'currency',
        currency: 'USD'
      });
      return formatter.format(value);
    }
  },
  watch: {
    transactions: {
      handler() {
        this.saveLists();
      },
      deep: true
    },
    lists: {
        handler() {
            localStorage.setItem('expense-tracker-lists', JSON.stringify(this.lists));
        },
        deep: true
    },
    activeList() {
        this.loadList();
    }
  },
  created() {
    const savedLists = localStorage.getItem('expense-tracker-lists');
    if (savedLists) {
      this.lists = JSON.parse(savedLists);
      const listKeys = Object.keys(this.lists);
      if (listKeys.length > 0) {
        this.activeList = listKeys[0];
        this.loadList();
      }
    }
  }
}
</script>

<style>
@media print {
  body * {
    visibility: hidden;
  }
  .printable, .printable * {
    visibility: visible;
  }
  .printable {
    position: absolute;
    left: 0;
    top: 0;
    width: 100%;
  }
}
</style>
<style scoped>
.expenses-tracker {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}
.list-manager, .transaction-form, .filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}
.transactions {
  display: flex;
  gap: 20px;
}
.income, .expenses-list {
  flex: 1;
}
ul {
  list-style: none;
  padding: 0;
}
li {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  border-bottom: 1px solid #ccc;
}
.summary {
  margin-top: 20px;
}
.btn {
    cursor: pointer;
    border: 1px solid #ccc;
    padding: 8px 12px;
    border-radius: 4px;
    background-color: #f0f0f0;
}
.btn-primary {
  background-color: #42b983;
  color: white;
  border-color: #42b983;
}
.btn-danger {
  background-color: #e74c3c;
  color: white;
  border-color: #e74c3c;
}
h2 {
  font-size: 1.2em;
}
h3 {
  font-size: 1em;
}
</style>