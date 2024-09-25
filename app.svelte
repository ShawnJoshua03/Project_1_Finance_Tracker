<script>
  import { onMount } from 'svelte';
  import { Chart, BarController, BarElement, CategoryScale, LinearScale, Title, Tooltip, Legend } from 'chart.js';

  // Register required components from Chart.js
  Chart.register(BarController, BarElement, CategoryScale, LinearScale, Title, Tooltip, Legend);

  // Preset expenses categories
  const PRESET_EXPENSES = [
    { label: 'Food', amount: 0, backgroundColor: 'rgba(54, 162, 235, 0.2)', borderColor: 'rgba(54, 162, 235, 1)' },
    { label: 'Rent/Utilities', amount: 0, backgroundColor: 'rgba(255, 159, 64, 0.2)', borderColor: 'rgba(255, 159, 64, 1)' },
    { label: 'Phone Bill', amount: 0, backgroundColor: 'rgba(75, 192, 192, 0.2)', borderColor: 'rgba(75, 192, 192, 1)' }
  ];

  // Investment categories
  const INVESTMENTS = [
    { label: 'Stocks', amount: 0, backgroundColor: 'rgba(75, 192, 192, 0.2)', borderColor: 'rgba(75, 192, 192, 1)' },
    { label: 'ETFs', amount: 0, backgroundColor: 'rgba(153, 102, 255, 0.2)', borderColor: 'rgba(153, 102, 255, 1)' },
    { label: 'Crypto', amount: 0, backgroundColor: 'rgba(255, 206, 86, 0.2)', borderColor: 'rgba(255, 206, 86, 1)' }
  ];

  let income = 0; // Total income
  let otherExpenses = [];
  let expenseDescription = '';
  let expenseAmount = '';
  let expenseCanvas, investmentCanvas;
  let expenseChart, investmentChart;

  // Function to add other expenses
  function addOtherExpense() {
    if (expenseDescription && expenseAmount) {
      otherExpenses.push({ description: expenseDescription, amount: parseFloat(expenseAmount), label: 'Other' });
      updateExpenseChart();
      expenseDescription = '';
      expenseAmount = '';
    }
  }

  // Function to add a preset expense
  function addPresetExpense(index, amount) {
    PRESET_EXPENSES[index].amount += parseFloat(amount);
    updateExpenseChart();
  }

  // Function to add an investment
  function addInvestment(index, amount) {
    INVESTMENTS[index].amount += parseFloat(amount);
    updateInvestmentChart();
  }

  // Calculate the total expenses
  function totalExpenses() {
    const presetTotal = PRESET_EXPENSES.reduce((total, expense) => total + expense.amount, 0);
    const otherTotal = otherExpenses.reduce((total, expense) => total + expense.amount, 0);
    return presetTotal + otherTotal;
  }

  // Update the expense chart
  function updateExpenseChart() {
    if (expenseChart) {
      const labels = ['Income', ...PRESET_EXPENSES.map(e => e.label), 'Other'];
      const expensesData = [
        income,
        ...PRESET_EXPENSES.map(e => e.amount),
        otherExpenses.reduce((total, expense) => total + expense.amount, 0)
      ];

      expenseChart.data.labels = labels;
      expenseChart.data.datasets[0].data = [income, ...expensesData.slice(1)];
      expenseChart.update(); // Update the chart
    }
  }

  // Update the investment chart
  function updateInvestmentChart() {
    if (investmentChart) {
      investmentChart.data.labels = INVESTMENTS.map(i => i.label);
      investmentChart.data.datasets[0].data = INVESTMENTS.map(i => i.amount);
      investmentChart.update(); // Update the chart with new data
    }
  }

  // Initialize charts on mount
  onMount(() => {
    // Expense chart initialization
    const maxExpenseAmount = Math.max(income, totalExpenses()) * 1.2 || 100;
    expenseChart = new Chart(expenseCanvas, {
      type: 'bar',
      data: {
        labels: [], // Will be updated dynamically
        datasets: [{
          label: 'Income & Expenses',
          data: [], // Will be updated dynamically
          backgroundColor: ['rgba(75, 192, 192, 0.2)', ...PRESET_EXPENSES.map(c => c.backgroundColor), 'rgba(153, 102, 255, 0.2)'],
          borderColor: ['rgba(75, 192, 192, 1)', ...PRESET_EXPENSES.map(c => c.borderColor), 'rgba(153, 102, 255, 1)'],
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            suggestedMax: maxExpenseAmount
          }
        }
      }
    });

    // Investment chart initialization
    const maxInvestmentAmount = Math.max(...INVESTMENTS.map(i => i.amount)) * 1.2 || 100;
    investmentChart = new Chart(investmentCanvas, {
      type: 'bar',
      data: {
        labels: [], // Will be updated dynamically
        datasets: [{
          label: 'Investments',
          data: [], // Will be updated dynamically
          backgroundColor: INVESTMENTS.map(i => i.backgroundColor),
          borderColor: INVESTMENTS.map(i => i.borderColor),
          borderWidth: 1
        }]
      },
      options: {
        responsive: true,
        scales: {
          y: {
            beginAtZero: true,
            suggestedMax: maxInvestmentAmount
          }
        }
      }
    });
  });
</script>

<main>
  <h1>Smart Expense & Investment Tracker</h1>

  <!-- Income Section -->
  <section>
    <h2>Set Your Income</h2>
    <input bind:value={income} type="number" placeholder="Enter Total Income" />
    <p>Total Income: ${income}</p>
  </section>

  <!-- Preset Expenses Section -->
  <section>
    <h2>Preset Expenses</h2>
    <div>
      {#each PRESET_EXPENSES as expense, index}
        <label for="preset-{index}">{expense.label}: $</label>
        <input id="preset-{index}" type="number" on:input={(e) => addPresetExpense(index, e.currentTarget.value)} placeholder={`Enter ${expense.label} amount`} />
      {/each}
    </div>
  </section>

  <!-- Other Expenses Section -->
  <section>
    <h2>Add Other Expenses</h2>
    <input bind:value={expenseDescription} placeholder="Other Expense Description" />
    <input bind:value={expenseAmount} type="number" placeholder="Other Expense Amount" />
    <button on:click={addOtherExpense}>Add Other Expense</button>
  </section>

  <!-- Investments Section -->
  <section>
    <h2>Investments</h2>
    <div>
      {#each INVESTMENTS as investment, index}
        <label for="investment-{index}">{investment.label}: $</label>
        <input id="investment-{index}" type="number" on:input={(e) => addInvestment(index, e.currentTarget.value)} placeholder={`Enter ${investment.label} amount`} />
      {/each}
    </div>
  </section>

  <!-- Total Expenses Display -->
  <section>
    <h3>Total Expenses: ${totalExpenses()}</h3>
  </section>

  <!-- Canvas for Expense Chart -->
  <section>
    <h3>Expense Chart</h3>
    <canvas bind:this={expenseCanvas} width="400" height="200"></canvas>
  </section>

  <!-- Canvas for Investment Chart -->
  <section>
    <h3>Investment Chart</h3>
    <canvas bind:this={investmentCanvas} width="400" height="200"></canvas>
  </section>
</main>

<style>
  main {
    max-width: 600px;
    margin: auto;
    padding: 1rem;
    font-family: Arial, sans-serif;
  }

  h1, h2 {
    color: #2c3e50;
    text-align: center;
  }

  section {
    margin: 2rem 0;
    padding: 1rem;
    border: 1px solid #ccc;
    border-radius: 5px;
    background: #f9f9f9;
    text-align: center;
  }

  input {
    margin: 0.5rem;
    padding: 0.5rem;
    width: calc(40% - 1rem);
  }

  button {
    padding: 0.5rem 1rem;
    margin-top: 0.5rem;
    cursor: pointer;
    background: #2c3e50;
    color: white;
    border: none;
    border-radius: 5px;
  }

  button:hover {
    background: #34495e;
  }

  canvas {
    margin-top: 1rem;
    width: 100%;
    height: 300px;
    background-color: lightgray;
  }
</style>
