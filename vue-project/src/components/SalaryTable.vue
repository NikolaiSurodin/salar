<script setup lang="ts">
import { ref, computed } from 'vue';
import html2pdf from 'html2pdf.js';

// Define the employee data structure
interface Employee {
  name: string;
  salary: number;
  netPay: number;
  cryptoAmount: number;
  lunchCompensation: number;
  total: number;
  totalCryptoEUR: number;
  rate: number;
  totalCryptoUSDT: number;
  wallet: string;
  paid: boolean;
}

// Input data
const inputData = ref<string>('');

// Parsed employees
const employees = ref<Employee[]>([]);

// Parse input data
const parseInputData = () => {
  if (!inputData.value.trim()) return;

  const lines = inputData.value.split('\n').filter(line => line.trim());
  const parsedEmployees: Employee[] = [];

  for (const line of lines) {
    const parts = line.split('\t').map(part => part.trim());
    if (parts.length < 11) continue;

    // Extract numeric values from strings
    const extractNumber = (str: string): number => {
      const match = str.match(/[\d\s,.]+/);
      if (!match) return 0;
      return parseFloat(match[0].replace(/\s/g, '').replace(',', '.'));
    };

    const salary = extractNumber(parts[1]);
    const netPay = extractNumber(parts[2]);
    const cryptoAmount = extractNumber(parts[3]);
    const lunchCompensation = extractNumber(parts[4]);
    const total = extractNumber(parts[6]);
    const totalCryptoEUR = extractNumber(parts[7]);
    const rate = extractNumber(parts[8]);
    const totalCryptoUSDT = extractNumber(parts[9]);

    parsedEmployees.push({
      name: parts[0],
      salary,
      netPay,
      cryptoAmount,
      lunchCompensation,
      total,
      totalCryptoEUR,
      rate,
      totalCryptoUSDT,
      wallet: parts[10],
      paid: parts[11]?.toLowerCase().includes('true') || false
    });
  }

  employees.value = parsedEmployees;
};

// Function to generate PDF
const generatePDF = () => {
  const element = document.getElementById('salary-cards');

  if (!element) {
    console.error('Element with id "salary-cards" not found');
    return;
  }

  const opt = {
    margin: 10,
    filename: `${employees.value[0]?.name}.pdf`,
    image: { type: 'jpeg', quality: 0.98 },
    html2canvas: { scale: 2 },
    jsPDF: { unit: 'mm', format: 'a4', orientation: 'portrait' }
  };

  html2pdf().set(opt).from(element).save();
};

// Format currency values
const formatCurrency = (value: number, currency: string = '€') => {
  return `${currency} ${value.toLocaleString('de-DE', { minimumFractionDigits: 2, maximumFractionDigits: 2 })}`;
};

// Format USDT values
const formatUSDT = (value: number) => {
  return `${value.toLocaleString('de-DE', { minimumFractionDigits: 2, maximumFractionDigits: 2 })} USDT`;
};

// Clear input data
const clearInputData = () => {
  inputData.value = '';
};

const date = ref('')

</script>

<template>
  <div class="app-container">
    <div class="sidebar">
      <div class="logo">
        <div class="logo-icon">B</div>
        <h3>Boom</h3>
      </div>
      <nav class="nav-menu">
        <div class="nav-item active">
          <span class="nav-icon">📊</span>
          <span>Salary Report</span>
        </div>
      </nav>
      <div class="user-profile">
        <div class="avatar">A</div>
        <div class="user-info">
          <div class="user-name">Alexandra</div>
          <div class="user-role">Sweety Administrator</div>
        </div>
      </div>
    </div>

    <div class="main-content">
      <header class="top-header">
        <h1>Salary Report</h1>
        <div class="header-actions">
          <div class="search-bar">
            <input type="text" placeholder="Search..." />
            <span class="search-icon">🔍</span>
          </div>
          <div class="notifications">
            <span class="notification-icon">🔔</span>
            <span class="notification-badge">3</span>
          </div>
        </div>
      </header>

      <div class="content-area">
        <div class="input-panel">
          <div class="panel-header">
            <h2>Paste Employee Data</h2>
            <button class="help-btn">?</button>
          </div>
          <p class="instruction">Copy and paste your employee data in the format shown below:</p>
          <pre class="example">Имя сотрудника	Оклад	Net pay	На крипту	Компенсация обедов		Итого	Итого на крипту, EUR	Rate	Итого на крипту, USDT	Wallet	Paid</pre>
          <div class="search-bar">
            <input type="text" placeholder="Дата" v-model="date" />
          </div>
          <textarea
            v-model="inputData"
            placeholder="Paste your data here..."
            class="data-input"
            rows="5"
          ></textarea>

          <div class="button-group">
            <button @click="parseInputData" class="secondary-btn">Create PDF</button>
            <button @click="clearInputData" class="secondary-btn">Clear</button>
            <button @click="generatePDF" class="action-btn">
              <span class="btn-icon">📄</span> Save PDF
            </button>
          </div>
        </div>
        <div id="salary-cards" class="salary-dashboard">
          <div v-if="employees.length === 0" class="empty-state">
            <div class="empty-icon">📝</div>
            <h3>No Data Available</h3>
            <p>Please paste your employee data above and click "Process Data" to generate the salary report.</p>
          </div>

          <template v-else>
            <div class="employee-grid">
              <div class="date">
                {{ date }}
              </div>
              <div class="employee-card">
                <div class="chevron-column">
                  <img src="/chevron.svg" alt="" class="chevron-item">
                  <img src="/chevron.svg" alt="" class="chevron-item">
                  <img src="/chevron.svg" alt="" class="chevron-item">
                  <img src="/chevron.svg" alt="" class="chevron-item">
                </div>
                <div class="card-content">
                  <div class="info-section" v-for="(em, index) in employees">
                    <div class="info-item column name-item">
                      <div class="info-label">Имя Сотрудника:</div>
                      <div class="info-value">{{ em.name }}</div>
                    </div>
                    <div class="info-item column address-item">
                      <div class="info-label">Адрес кошелька</div>
                      <div class="info-value">{{ em.wallet }}</div>
                    </div>
                    <div class="info-item">
                      <div class="info-label">Оклад</div>
                      <div class="info-value">€{{ em.salary }}</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">На карту:</div>
                      <div class="info-value">-</div>
                    </div>
                    <div class="info-item">
                      <div class="info-label">На кошелек:</div>
                      <div class="info-value">€ 326,47</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">Компенсация обедов:</div>
                      <div class="info-value">€ {{ em.lunchCompensation }}</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">Итого:</div>
                      <div class="info-value">€ {{ em.total }}</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">Итого на кошелек, EUR:</div>
                      <div class="info-value">€ {{ em.totalCryptoEUR }}</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">Rate:</div>
                      <div class="info-value">{{ em.rate }}</div>
                    </div>

                    <div class="info-item">
                      <div class="info-label">Итого на кошелек, USDT:</div>
                      <div class="info-value">{{ em.totalCryptoUSDT }} USDT</div>
                    </div>
                  </div>
                </div>
              </div>
              <div class="lock-img">
                <img src="/lock.svg" alt="">
              </div>
            </div>
          </template>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Inter:ital,opsz,wght@0,14..32,100..900;1,14..32,100..900&family=Space+Mono&display=swap');
/* Base Styles */
:root {
  --primary-color: #4361ee;
  --secondary-color: #3a0ca3;
  --accent-color: #f72585;
  --success-color: #4cc9f0;
  --warning-color: #f77f00;
  --danger-color: #e63946;
  --light-color: #f8f9fa;
  --dark-color: #212529;
  --gray-color: #6c757d;
  --border-radius: 10px;
  --box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  --transition: all 0.3s ease;
}

* {
  box-sizing: border-box;
  margin: 0;
  padding: 0;
}

/* Layout */
.app-container {
  display: flex;
  min-height: 100vh;
  font-family: 'Poppins', 'Arial', sans-serif;
  color: var(--dark-color);
  background-color: #f5f7fa;
}

.date {
  font-family: Inter;
  font-weight: 600;
  font-style: Italic;
  font-size: 28px;
  leading-trim: NONE;
  line-height: 20px;
  letter-spacing: 0%;
  text-align: right;
  margin-right: 20px;
}

/* Sidebar */
.sidebar {
  width: 280px;
  background: linear-gradient(to bottom, var(--primary-color), var(--secondary-color));
  color: var(--gray-color);
  padding: 20px;
  display: flex;
  flex-direction: column;
  position: fixed;
  height: 100vh;
  z-index: 10;
}

.logo {
  display: flex;
  align-items: center;
  margin-bottom: 40px;
  padding: 10px 0;
}

.logo-icon {
  width: 40px;
  height: 40px;
  background-color: white;
  color: var(--primary-color);
  border-radius: 10px;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  font-size: 20px;
  margin-right: 15px;
}

.logo h3 {
  font-size: 20px;
  font-weight: 600;
}

.nav-menu {
  flex: 1;
}

.nav-item {
  display: flex;
  align-items: center;
  padding: 15px;
  margin-bottom: 10px;
  border-radius: var(--border-radius);
  cursor: pointer;
  transition: var(--transition);
}

.nav-item:hover {
  background-color: rgba(255, 255, 255, 0.1);
}

.nav-item.active {
  background-color: rgba(255, 255, 255, 0.2);
}

.nav-icon {
  margin-right: 15px;
  font-size: 18px;
}

.user-profile {
  display: flex;
  align-items: center;
  padding: 15px;
  border-top: 1px solid rgba(255, 255, 255, 0.1);
  margin-top: 20px;
}

.avatar {
  width: 40px;
  height: 40px;
  background-color: white;
  color: var(--primary-color);
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-weight: bold;
  margin-right: 15px;
}

.user-info {
  flex: 1;
}

.user-name {
  font-weight: 600;
  font-size: 14px;
}

.user-role {
  font-size: 12px;
  opacity: 0.8;
}

/* Main Content */
.main-content {
  flex: 1;
  margin-left: 280px;
  padding: 20px;
}

.top-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding-bottom: 20px;
  border-bottom: 1px solid #e9ecef;
}

.top-header h1 {
  font-size: 28px;
  font-weight: 700;
  color: var(--dark-color);
}

.header-actions {
  display: flex;
  align-items: center;
}

.search-bar {
  position: relative;
  margin-right: 20px;
}

.search-bar input {
  padding: 10px 15px;
  padding-right: 40px;
  border: 1px solid #e9ecef;
  border-radius: 20px;
  width: 250px;
  font-size: 14px;
}

.search-icon {
  position: absolute;
  right: 15px;
  top: 50%;
  transform: translateY(-50%);
}

.notifications {
  position: relative;
}

.notification-icon {
  font-size: 20px;
  cursor: pointer;
}

.notification-badge {
  position: absolute;
  top: -8px;
  right: -8px;
  background-color: var(--danger-color);
  color: white;
  border-radius: 50%;
  width: 18px;
  height: 18px;
  font-size: 12px;
  display: flex;
  align-items: center;
  justify-content: center;
}

/* Content Area */
.content-area {
  display: flex;
  flex-direction: column;
  gap: 30px;
}

/* Input Panel */
.input-panel {
  background-color: white;
  border-radius: var(--border-radius);
  padding: 25px;
  box-shadow: var(--box-shadow);
}

.panel-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.panel-header h2 {
  font-size: 20px;
  font-weight: 600;
  color: var(--dark-color);
}

.help-btn {
  width: 30px;
  height: 30px;
  border-radius: 50%;
  background-color: var(--light-color);
  border: none;
  color: var(--gray-color);
  font-weight: bold;
  cursor: pointer;
  transition: var(--transition);
}

.help-btn:hover {
  background-color: var(--primary-color);
  color: white;
}

.instruction {
  margin-bottom: 15px;
  color: var(--gray-color);
  font-size: 14px;
}

.example {
  background-color: #f8f9fa;
  padding: 15px;
  border-radius: var(--border-radius);
  font-size: 13px;
  font-family: monospace;
  overflow-x: auto;
  margin-bottom: 20px;
  white-space: pre-wrap;
  word-break: break-word;
  border-left: 4px solid var(--primary-color);
}

.data-input {
  width: 100%;
  padding: 15px;
  border: 1px solid #e9ecef;
  border-radius: var(--border-radius);
  font-family: monospace;
  resize: vertical;
  margin-bottom: 20px;
  transition: var(--transition);
}

.data-input:focus {
  outline: none;
  border-color: var(--primary-color);
  box-shadow: 0 0 0 3px rgba(67, 97, 238, 0.1);
}

.button-group {
  display: flex;
  gap: 15px;
}

.primary-btn, .secondary-btn, .action-btn {
  padding: 12px 25px;
  border: none;
  border-radius: var(--border-radius);
  font-weight: 600;
  cursor: pointer;
  transition: var(--transition);
  display: flex;
  align-items: center;
  justify-content: center;
}

.primary-btn {
  background-color: var(--primary-color);
  color: black;
}

.primary-btn:hover {
  background-color: var(--secondary-color);
  transform: translateY(-2px);
}

.secondary-btn {
  background-color: var(--light-color);
  color: var(--dark-color);
}

.secondary-btn:hover {
  background-color: #e9ecef;
}

.btn-icon {
  margin-right: 8px;
}

/* Salary Dashboard */
.salary-dashboard {
  background-color: white;
  border-radius: var(--border-radius);
  padding: 25px;
  box-shadow: var(--box-shadow);
}

.empty-state {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  padding: 60px 20px;
  text-align: center;
}

.empty-icon {
  font-size: 50px;
  margin-bottom: 20px;
  color: var(--gray-color);
}

.empty-state h3 {
  font-size: 20px;
  margin-bottom: 10px;
  color: var(--dark-color);
}

.empty-state p {
  color: var(--gray-color);
  max-width: 500px;
}

.dashboard-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 25px;
}

.dashboard-header h2 {
  font-size: 20px;
  font-weight: 600;
  color: var(--dark-color);
}

.dashboard-actions {
  display: flex;
  gap: 15px;
}

.action-btn {
  background-color: var(--light-color);
  color: var(--dark-color);
  padding: 10px 15px;
  font-size: 14px;
}

.action-btn:hover {
  background-color: #e9ecef;
}

/* Employee Grid */
.employee-grid {
  position: relative;
  max-width: 662px;
  margin: 0 auto;
  overflow: hidden;
}

.employee-card {
  display: flex;
  justify-content: center;
  gap: 20px;
  padding-top: 108px;
  padding-bottom: 60px;
  background-image: url(/ellipse.svg);
  background-position-y: -12px;
  background-repeat: no-repeat;
  background-position: center;
  background-size: 100%;
  overflow: hidden;

}

.chevron-column {
  position: absolute;
  top: 0;
  left: 0;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.chevron-item {
  width: 100%;
  height: 100%;
}

.card-header h3 {
  font-size: 18px;
  font-weight: 600;
  color: var(--dark-color);
}

.card-content {
  width: 336px;
}

.info-section {
  display: flex;
  flex-direction: column;
  gap: 10px;
  z-index: 1;
  position: relative;
}

.info-group {
  //display: flex;
  //gap: 20px;
  //margin-bottom: 15px;
}

.info-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
}

.name-item {
  margin-bottom: 40px;
}

.address-item {
  margin-bottom: 70px;
}

.column {
  flex-direction: column;
  gap: 10px;
  border-bottom: 1px solid black;
  align-items: flex-start;
}

.info-item.total {
  //margin-top: 15px;
  //padding-top: 15px;
  //border-top: 1px dashed #e9ecef;
}

.info-label {
  font-family: 'Inter';
  font-weight: 600;
  font-size: 16px;
  line-height: 20px;
  letter-spacing: 0%;
}

.info-value {
  font-family: Inter;
  font-weight: 400;
  font-size: 14px;
  line-height: 16px;
  letter-spacing: 0%;
}

.info-item.highlight .info-value {
  color: var(--accent-color);
  font-size: 18px;
}

.wallet {
  font-family: monospace;
  font-size: 14px;
  word-break: break-all;
}

.crypto-section {
  background-color: #f8f9fa;
  padding: 20px;
  border-radius: var(--border-radius);
  margin-bottom: 20px;
}

.section-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.section-header h4 {
  font-size: 16px;
  font-weight: 600;
  color: var(--dark-color);
}

.crypto-icon {
  font-size: 20px;
  color: var(--accent-color);
}

.card-actions {
  display: flex;
  gap: 10px;
}

.card-btn {
  flex: 1;
  padding: 10px;
  border: 1px solid #e9ecef;
  border-radius: var(--border-radius);
  background-color: white;
  color: var(--dark-color);
  font-weight: 500;
  cursor: pointer;
  transition: var(--transition);
}

.card-btn:hover {
  background-color: var(--primary-color);
  color: white;
  border-color: var(--primary-color);
}

.lock-img {
  position: absolute;
  right: 0;
  bottom: 0;
  opacity: 0.8;
}

/* Responsive adjustments */
@media (max-width: 1200px) {
  .employee-grid {
    grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  }
}

@media (max-width: 992px) {
  .sidebar {
    width: 80px;
    padding: 15px 10px;
  }

  .logo h3, .user-info, .nav-item span:not(.nav-icon) {
    display: none;
  }

  .logo-icon {
    margin-right: 0;
  }

  .nav-icon {
    margin-right: 0;
    font-size: 22px;
  }

  .nav-item {
    justify-content: center;
  }

  .avatar {
    margin-right: 0;
  }

  .user-profile {
    justify-content: center;
  }

  .main-content {
    margin-left: 80px;
  }
}

@media (max-width: 768px) {
  .top-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .header-actions {
    margin-top: 15px;
    width: 100%;
  }

  .search-bar {
    width: 100%;
    margin-right: 15px;
  }

  .search-bar input {
    width: 100%;
  }

  .employee-grid {
    grid-template-columns: 1fr;
  }

  .info-group {
    flex-direction: column;
    gap: 15px;
  }

  .chevron-column {
    display: none;
  }

  .employee-card {
    padding-left: 70px;
  }
}

@media (max-width: 576px) {
  .button-group {
    flex-direction: column;
  }

  .dashboard-header {
    flex-direction: column;
    align-items: flex-start;
  }

  .dashboard-actions {
    margin-top: 15px;
    width: 100%;
  }
}
</style>
