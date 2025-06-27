<template>
  <div class="print-page" id="printArea">
    <!-- Header -->
    <div class="header">
      <h2>বাখরাবাদ গ্যাস ডিস্ট্রিবিউশন কোম্পানি লিমিটেড</h2>
      <h4>(পেট্রোবাংলার একটি প্রতিষ্ঠান)</h4>
      <h5>প্রধান কার্যালয়, চাঁপাপুর, কুমিল্লা।</h5>
      <div class="header-row">
        <div>টীম নং: <span contenteditable="true">_________</span></div>
        <div>{{ currentDate }}</div>
      </div>
      <hr />
    </div>

    <!-- Table -->
    <table class="report-table">
      <thead>
        <tr>
          <th>ক্রমিক</th>
          <th>গ্রাহকের নাম ও ঠিকানা</th>
          <th>গ্রাহক কোড</th>
          <th>অনুমোদিত বার্নার</th>
          <th>প্রাপ্ত বার্নার</th>
          <th>বকেয়া বিবরণ</th>
          <th>মন্তব্য</th>
        </tr>
      </thead>
      <tbody>
        <tr v-for="(entry, index) in entries" :key="index">
          <td>{{ index + 1 }}</td>
          <td>
            {{ entry.customerName }} <br />
            {{ entry.customerAddress }} <br />
            <span v-if="entry.location">
              লোকেশন: {{ entry.location.latitude.toFixed(7) }}, {{ entry.location.longitude.toFixed(7) }}
            </span>
          </td>
          <td>{{ entry.customerCode }}</td>
          <td>{{ entry.approvedBurner }}</td>
          <td>{{ entry.burnerFound }}</td>
          <td>{{ entry.due || 'নেই' }}</td>
          <td>{{ entry.remarks || 'নেই' }}</td>
        </tr>
      </tbody>
    </table>

    <!-- Signature -->
    <div class="signature-box">
      <div>পরিদর্শনকারী কর্মকর্তার স্বাক্ষর</div>
      <div class="signature-line"></div>
    </div>

    <!-- Print Button -->
    <div class="no-print text-center mt-3">
      <button class="btn btn-primary" @click="printPage">
        প্রিন্ট করুন
      </button>
    </div>
  </div>
</template>

<script>
export default {
  name: "BGDCLPrintableReport",
  props: {
    entries: {
      type: Array,
      required: true
    }
  },
  data() {
    return {
      currentDate: new Date().toLocaleDateString('bn-BD', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    };
  },
  methods: {
    printPage() {
      window.print();
    }
  }
};
</script>

<style>
@import url('https://fonts.googleapis.com/css2?family=Noto+Sans+Bengali&display=swap');

.print-page {
  font-family: 'Noto Sans Bengali', sans-serif;
  padding: 20px;
  background: white;
  color: black;
}

.header {
  text-align: center;
}

.header-row {
  display: flex;
  justify-content: space-between;
  margin-top: 10px;
}

.report-table {
  width: 100%;
  border-collapse: collapse;
  margin-top: 20px;
  page-break-inside: auto;
}

.report-table th,
.report-table td {
  border: 1px solid #000;
  padding: 8px;
  text-align: center;
  vertical-align: top;
  page-break-inside: avoid;
}

.signature-box {
  margin-top: 50px;
  text-align: center;
}

.signature-line {
  width: 200px;
  border-bottom: 1px solid #000;
  margin: 10px auto 0 auto;
  height: 30px;
}

.no-print {
  margin-top: 20px;
}

@media print {
  .no-print {
    display: none;
  }
  body {
    -webkit-print-color-adjust: exact;
    print-color-adjust: exact;
  }
  .report-table {
    page-break-after: auto;
  }
  .report-table tr {
    page-break-inside: avoid;
    page-break-after: auto;
  }
}
</style>
