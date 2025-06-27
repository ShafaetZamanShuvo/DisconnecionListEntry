<template>
  <div>
    <!-- Print Button -->
    <button @click="window.print()" class="no-print btn btn-success mb-3">
      <i class="fas fa-print"></i> প্রিন্ট করুন
    </button>

    <!-- Paginated Print Pages -->
    <div
      v-for="(pageEntries, pageIndex) in paginatedEntries"
      :key="pageIndex"
      class="print-page"
    >
      <!-- Header -->
      <div class="print-header">
        <h3>বিজিডিসিএল ফিল্ড রিপোর্ট</h3>
        <div>টীম নং: {{ teamNo }} | তারিখ: {{ currentDate }}</div>
      </div>

      <!-- Table -->
      <table class="print-table">
        <thead>
          <tr>
            <th>ক্রমিক</th>
            <th>কোড</th>
            <th>নাম, ঠিকানা ও অবস্থান</th>
            <th>অনুমোদিত বার্নার</th>
            <th>পাওয়া বার্নার</th>
            <th>বকেয়া</th>
            <th>মন্তব্য</th>
          </tr>
        </thead>
        <tbody>
          <tr v-for="(entry, idx) in pageEntries" :key="idx">
            <td>{{ idx + 1 + pageIndex * 15 }}</td>
            <td>{{ entry.customerCode }}</td>
            <td>
              {{ entry.customerName }}<br />
              {{ entry.customerAddress }}<br />
              <span v-if="entry.location">
                {{ entry.location.latitude.toFixed(7) }}, {{ entry.location.longitude.toFixed(7) }}
              </span>
            </td>
            <td>{{ entry.approvedBurner }}</td>
            <td>{{ entry.burnerFound }}</td>
            <td>{{ entry.due || 'নেই' }}</td>
            <td>{{ entry.remarks || 'নেই' }}</td>
          </tr>
          <!-- Fill empty rows to maintain 15 rows per page -->
          <tr v-for="n in 15 - pageEntries.length" :key="'empty-' + n">
            <td colspan="7" style="height: 25px;">&nbsp;</td>
          </tr>
        </tbody>
      </table>

      <!-- Footer -->
      <div class="print-footer">
        <p>স্বাক্ষর:</p>
      </div>

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
      return new Date().toLocaleDateString("bn-BD");
    },
    paginatedEntries() {
      const chunkSize = 15;
      const chunks = [];
      for (let i = 0; i < this.entries.length; i += chunkSize) {
        chunks.push(this.entries.slice(i, i + chunkSize));
      }
      return chunks.length > 0 ? chunks : [[]];
    },
  },
};
</script>

<style>
.print-page {
  width: 210mm;
  min-height: 297mm;
  padding: 10mm;
  box-sizing: border-box;
  font-family: 'Noto Sans Bengali', sans-serif;
}

.print-header,
.print-footer {
  text-align: center;
  margin-bottom: 10px;
}

.print-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
}

.print-table th,
.print-table td {
  border: 1px solid #000;
  padding: 4px;
  text-align: left;
}

.page-break {
  page-break-after: always;
}

@media print {
  .no-print {
    display: none;
  }
}
</style>