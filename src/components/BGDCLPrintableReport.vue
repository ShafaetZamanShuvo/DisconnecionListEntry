<template>
  <div class="report-container">
    <!-- Action Buttons -->
    <div class="no-print action-buttons">
      <button @click="goBack" class="btn btn-secondary">
        <i class="fas fa-arrow-left"></i> ফিরে যান
      </button>
      <button @click="printReport" class="btn btn-success">
        <i class="fas fa-print"></i> প্রিন্ট করুন
      </button>
    </div>

    <!-- Paginated Print Pages -->
    <div
      v-for="(pageEntries, pageIndex) in paginatedEntries"
      :key="pageIndex"
      class="print-page"
    >
      <!-- Header -->
      <div class="print-header">
        <div class="header-content">
          <h2 class="company-name">বাখরাবাদ গ্যাস ডিস্ট্রিবিউশন কোম্পানী লিমিটেড</h2>
          <p class="company-address">(পেট্রোবাংলার একটি কোম্পানি)</p>
          <h3 class="report-title">ফিল্ড সার্ভে রিপোর্ট</h3>
          <div class="report-info">
            <span class="team-info">
              <strong>টীম নং:</strong> {{ teamNo }} | 
              <strong>তারিখ:</strong> {{ currentDate }} | 
              <strong>পৃষ্ঠা:</strong> {{ pageIndex + 1 }} / {{ paginatedEntries.length }}
            </span>
          </div>
        </div>
      </div>

      <!-- Decorative Line -->
      <div class="decorative-line"></div>

      <!-- Table -->
      <table class="print-table">
        <thead>
          <tr class="table-header">
            <th class="col-serial">ক্রমিক</th>
            <th class="col-code">কোড</th>
            <th class="col-details">নাম, ঠিকানা ও অবস্থান</th>
            <th class="col-approved">অনুমোদিত বার্নার</th>
            <th class="col-found">পাওয়া বার্নার</th>
            <th class="col-due">বকেয়া</th>
            <th class="col-remarks">মন্তব্য</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(entry, idx) in pageEntries" :key="idx" class="data-row">
            <td class="serial-cell">{{ idx + 1 + pageIndex * 15 }}</td>
            <td class="code-cell">{{ entry.customerCode }}</td>
            <td class="details-cell">
              <div class="customer-info">
                <div class="customer-name">{{ entry.customerName }}</div>
                <div class="customer-address">{{ entry.customerAddress }}</div>
                <div v-if="entry.location" class="location-info">
                  <i class="fas fa-map-marker-alt"></i>
                  {{ entry.location.latitude.toFixed(6) }}, {{ entry.location.longitude.toFixed(6) }}
                </div>
              </div>
            </td>
            <td class="approved-cell">{{ entry.approvedBurner }}</td>
            <td class="found-cell">{{ entry.burnerFound }}</td>
            <td class="due-cell">
              <span v-if="entry.due && entry.due !== '0'" class="due-amount">৳{{ entry.due }}</span>
              <span v-else class="no-due">নেই</span>
            </td>
            <td class="remarks-cell">{{ entry.remarks || 'নেই' }}</td>
          </tr>
          
          <!-- Fill empty rows to maintain 15 rows per page -->
          <tr v-for="n in Math.max(0, 15 - pageEntries.length)" :key="'empty-' + n" class="empty-row">
            <td>&nbsp;</td>
            <td>&nbsp;</td>
            <td>&nbsp;</td>
            <td>&nbsp;</td>
            <td>&nbsp;</td>
            <td>&nbsp;</td>
            <td>&nbsp;</td>
          </tr>
        </tbody>
      </table>

      <!-- Summary Section -->
      <div class="summary-section">
        <div class="summary-item">
          <strong>মোট পরিদর্শন:</strong> {{ entries.length }}
        </div>
        <div class="summary-item">
          <strong>মোট বকেয়া আদায়:</strong> ৳{{ totalDue }}
        </div>
      </div>

      <!-- Footer -->
      <div class="print-footer">
        <div class="signature-section">
          <div class="signature-box">
            <div class="signature-line"></div>
            <p class="signature-label">তথ্য সংগ্রহকারীর স্বাক্ষর</p>
          </div>
          <div class="signature-box">
            <div class="signature-line"></div>
            <p class="signature-label">সুপারভাইজারের স্বাক্ষর</p>
          </div>
        </div>
        <div class="footer-note">
          <p>এই রিপোর্ট {{ new Date().toLocaleString('bn-BD') }} তারিখে তৈরি করা হয়েছে।</p>
        </div>
      </div>

      <!-- Page Break -->
      <div v-if="pageIndex < paginatedEntries.length - 1" class="page-break"></div>
    </div>
  </div>
</template>

<script>
export default {
  name: "BGDCLPrintableReport",
  props: {
    entries: {
      type: Array,
      required: true,
    },
    teamNo: {
      type: String,
      default: "____",
    },
  },
  computed: {
    currentDate() {
      return new Date().toLocaleDateString("bn-BD", {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      });
    },
    paginatedEntries() {
      const chunkSize = 15;
      const chunks = [];
      for (let i = 0; i < this.entries.length; i += chunkSize) {
        chunks.push(this.entries.slice(i, i + chunkSize));
      }
      return chunks.length > 0 ? chunks : [[]];
    },
    totalDue() {
      return this.entries.reduce((total, entry) => {
        return total + (parseFloat(entry.due) || 0);
      }, 0).toFixed(2);
    }
  },
  methods: {
    printReport() {
      window.print();
    },
    goBack() {
      this.$emit('close');
    }
  },
};
</script>

<style scoped>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Bengali:wght@300;400;600;700&display=swap');

.report-container {
  font-family: 'Noto Sans Bengali', 'Arial', sans-serif;
  background-color: #f8f9fa;
  min-height: 100vh;
}

.action-buttons {
  position: fixed;
  top: 20px;
  right: 20px;
  z-index: 1000;
  display: flex;
  gap: 10px;
}

.btn {
  padding: 10px 20px;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s ease;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
}

.btn-secondary {
  background: linear-gradient(135deg, #6c757d, #495057);
  color: white;
}

.btn-success {
  background: linear-gradient(135deg, #28a745, #20c997);
  color: white;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0,0,0,0.2);
}

.print-page {
  width: 210mm;
  min-height: auto;
  padding: 15mm;
  margin: 20px auto;
  background: white;
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
  box-sizing: border-box;
  position: relative;
  break-inside: avoid;
  border: 1px solid #dee2e6;
}

.print-header {
  display: flex;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 3px solid #007bff;
}

.header-content {
  flex: 1;
}

.company-name {
  font-size: 20px;
  font-weight: 700;
  color: #2c3e50;
  margin: 0 0 5px 0;
  line-height: 1.2;
}

.report-title {
  font-size: 16px;
  font-weight: 600;
  color: #007bff;
  margin: 0 0 8px 0;
}

.report-info {
  font-size: 12px;
  color: #6c757d;
}

.team-info {
  background: #f8f9fa;
  padding: 5px 10px;
  border-radius: 20px;
  border: 1px solid #dee2e6;
}

.decorative-line {
  height: 3px;
  background: linear-gradient(90deg, #007bff, #28a745, #ffc107);
  margin-bottom: 15px;
  border-radius: 2px;
}

.print-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 11px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0,0,0,0.1);
  border-radius: 8px;
  overflow: hidden;
}

.table-header th {
  background-color: #343a40;
  color: white;
  padding: 12px 8px;
  text-align: center;
  font-weight: 600;
  border-right: 1px solid rgba(255, 255, 255, 0.2);
  border-radius: 0;
}

.table-header th:first-child {
  border-radius: 8px 0 0 0;
}

.table-header th:last-child {
  border-right: none;
}

.col-serial { width: 8%; }
.col-code { width: 12%; }
.col-details { width: 30%; }
.col-approved { width: 12%; }
.col-found { width: 12%; }
.col-due { width: 10%; }
.col-remarks { width: 16%; }

.data-row {
  border-bottom: 1px solid #dee2e6;
  transition: background-color 0.2s ease;
}

.data-row:hover {
  background-color: #f8f9fa;
}

.data-row:nth-child(even) {
  background-color: #fbfbfb;
}

.print-table td {
  padding: 8px;
  border-right: 1px solid #dee2e6;
  vertical-align: top;
}

.print-table td:last-child {
  border-right: none;
}

.serial-cell, .code-cell, .approved-cell, .found-cell {
  text-align: center;
  font-weight: 600;
}

.details-cell {
  line-height: 1.4;
}

.customer-name {
  font-weight: 600;
  color: #2c3e50;
  margin-bottom: 3px;
}

.customer-address {
  color: #6c757d;
  font-size: 10px;
  margin-bottom: 3px;
}

.location-info {
  color: #007bff;
  font-size: 9px;
  font-weight: 500;
}

.location-info i {
  margin-right: 2px;
}

.due-amount {
  color: #dc3545;
  font-weight: 600;
}

.no-due {
  color: #28a745;
  font-style: italic;
}

.empty-row {
  height: 20px !important;
  background-color: transparent !important;
}

.summary-section {
  display: flex;
  justify-content: space-between;
  background: linear-gradient(135deg, #f8f9fa, #e9ecef);
  padding: 10px;
  border-radius: 8px;
  margin-bottom: 20px;
  border: 1px solid #dee2e6;
  font-size: 12px;
}

.summary-item {
  font-weight: 600;
  color: #2c3e50;
}

.print-footer {
  margin-top: auto;
  border-top: 2px solid #dee2e6;
  padding-top: 10px;
  font-size: 12px;
}

.signature-section {
  display: flex;
  justify-content: space-between;
  margin-bottom: 15px;
}

.signature-box {
  text-align: center;
  width: 45%;
}

.signature-line {
  height: 1px;
  background-color: #000;
  margin-bottom: 5px;
  margin-top: 30px;
}

.signature-label {
  font-size: 12px;
  color: #6c757d;
  margin: 0;
  font-weight: 500;
}

.footer-note {
  text-align: center;
  font-size: 10px;
  color: #6c757d;
  font-style: italic;
  border-top: 1px solid #dee2e6;
  padding-top: 10px;
}

.page-break {
  page-break-after: always;
}

/* Print Styles */
@media print {
  @page {
    size: A4;
    margin: 0;
  }

  body, html {
    margin: 0;
    padding: 0;
  }

  .no-print {
    display: none !important;
  }
  
  .report-container {
    background: white;
  }
  
  .print-page {
    margin: 0;
    padding: 10mm 8mm;
    box-shadow: none;
    min-height: auto;
    height: auto;
    page-break-after: always;
    break-inside: avoid;
    overflow: visible;
    border: 1px solid #ccc;
  }
  
  .print-page:last-child {
    page-break-after: auto;
  }
  
  .empty-row {
    height: 20px !important;
    background-color: transparent !important;
  }
  
  .summary-section,
  .print-footer {
    padding: 5px 10px;
    font-size: 12px;
  }
  
  .data-row:hover {
    background-color: transparent !important;
  }
}

/* Responsive Design */
@media (max-width: 768px) {
  .action-buttons {
    position: relative;
    top: auto;
    right: auto;
    justify-content: center;
    margin-bottom: 20px;
  }
  
  .print-page {
    width: 100%;
    margin: 10px;
    padding: 10mm;
  }
  
  .print-table {
    font-size: 10px;
  }
  
  .company-name {
    font-size: 16px;
  }
  
  .report-title {
    font-size: 14px;
  }
  
  .header-content {
    text-align: center;
  }
  
  .print-header {
    flex-direction: column;
    text-align: center;
  }
}
</style>
