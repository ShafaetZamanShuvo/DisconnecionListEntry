<template>
  <div class="report-issue-container">
    <!-- Action Buttons -->
    

    <!-- Issue Entry Form -->
    <div class="issue-form">
      <h4>ডিসকানেকশন রিপোর্ট এন্ট্রি</h4>
      <div v-for="(field, key) in fields" :key="key" class="mb-2">
        <label class="form-label">{{ field.label }}</label>

        <!-- Location Field (Special handling for lat/long) -->
        <div v-if="key === 'location'" class="location-inputs">
          <div class="row">
            <div class="col-6">
              <label class="form-label-sm">অক্ষাংশ (Latitude)</label>
              <input
                v-model="entryLocal[key].latitude"
                type="number"
                step="any"
                class="form-control form-control-sm"
                placeholder="23.8103"
              />
            </div>
            <div class="col-6">
              <label class="form-label-sm">দ্রাঘিমাংশ (Longitude)</label>
              <input
                v-model="entryLocal[key].longitude"
                type="number"
                step="any"
                class="form-control form-control-sm"
                placeholder="90.4125"
              />
            </div>
          </div>
        </div>

        <!-- Select Field -->
        <select
          v-else-if="field.type === 'select'"
          v-model="entryLocal[key]"
          class="form-select form-select-sm"
        >
          <option value="">নির্বাচন করুন</option>
          <option v-for="option in field.options" :key="option" :value="option">
            {{ option }}
          </option>
        </select>

        <!-- Multi Select Field -->
        <select
          v-else-if="field.type === 'multiselect'"
          v-model="entryLocal[key]"
          class="form-select form-select-sm"
          multiple
        >
          <option v-for="option in field.options" :key="option" :value="option">
            {{ option }}
          </option>
        </select>

        <!-- Date/Time Field -->
        <input
          v-else-if="field.type === 'datetime-local'"
          type="datetime-local"
          v-model="entryLocal[key]"
          class="form-control form-control-sm"
        />

        <!-- Textarea -->
        <textarea
          v-else-if="field.type === 'textarea'"
          v-model="entryLocal[key]"
          class="form-control form-control-sm"
          rows="2"
        ></textarea>

        <!-- Text Input -->
        <input
          v-else-if="field.type === 'text'"
          v-model="entryLocal[key]"
          class="form-control form-control-sm"
        />
      </div>

      <!-- Header Information -->
      <div class="header-info mt-3">
        <div class="row">
          <div class="col-6">
            <label class="form-label">তারিখ</label>
            <input
              v-model="headerInfo.date"
              type="date"
              class="form-control form-control-sm"
            />
          </div>
          <div class="col-6">
            <label class="form-label">টিম নং</label>
            <input
              v-model="headerInfo.teamNo"
              type="text"
              class="form-control form-control-sm"
              placeholder="টিম-০১"
            />
          </div>
        </div>
      </div>
    </div>
    <div class="no-print action-buttons mt-5">
      <button @click="goBack" class="btn btn-secondary btn-sm">
        <i class="fas fa-arrow-left"></i> ফিরে যান
      </button>
      <button
        @click="generateDocx"
        class="btn btn-success btn-sm"
        style="margin-left: 10px"
      >
        <i class="fas fa-print"></i> প্রিন্ট করুন
      </button>
    </div>
  </div>
</template>

<script setup>
import { Document, Packer, Paragraph, TextRun, AlignmentType } from "docx";
import { saveAs } from "file-saver";
import { ref } from "vue";

const props = defineProps({
  entry: { type: Object, required: true },
  index: { type: Number, required: true },
});

const emit = defineEmits(["close"]);

// Initialize entry with proper location structure
const entryLocal = ref({ 
  ...props.entry,
  location: props.entry.location || { latitude: '', longitude: '' }
});

// Header information
const headerInfo = ref({
  date: new Date().toISOString().split('T')[0],
  teamNo: ''
});

// Define fields in Bangla with type and options
const fields = {
  customerCode: { label: "গ্রাহক সংকেত নং", type: "text" },
  customerName: { label: "গ্রাহকের নাম", type: "textarea" },
  customerAddress: { label: "গ্রাহকের ঠিকানা", type: "textarea" },
  location: { label: "অবস্থান (অক্ষাংশ ও দ্রাঘিমাংশ)", type: "location" },
  approvedBurner: { label: "অনুমোদিত চুলা", type: "text" },
  burnerFound: { label: "প্রাপ্ত চুলা", type: "text" },
  meterType: { label: "মিটারের ধরন", type: "text" },
  meterNo: { label: "মিটার নং", type: "text" },
  meterReading: { label: "মিটারের পাঠ", type: "text" },
  due: { label: "বকেয়া টাকা", type: "text" },
  meterCondition: {
    label: "সংযোগ বিচ্ছিন্নকালীন মিটারের অবস্থা",
    type: "select",
    options: ["ঠিক", "ভাঙা", "কার্যক্ষমতাহীন"],
  },
  indexCondition: {
    label: "সংযোগ বিচ্ছিন্নকালীন ইনডেক্স অবস্থা",
    type: "select",
    options: ["ঠিক", "ভাঙা", "কুয়াশাচ্ছন্ন", "সন্দেহজনক"],
  },
  disconnectDateTime: { label: "বিচ্ছিন্নকরণের তারিখ ও সময়", type: "datetime-local" },
  removedEquipment: {
    label: "অপসারণকৃত সরঞ্জাম",
    type: "multiselect",
    options: ["মিটার", "রেগুলেটর", "সেন্সিং পাইপ"],
  },
  disconnectType: {
    label: "বিচ্ছিন্ন করণের ধরন",
    type: "multiselect",
    options: [
      "স্থায়ী",
      "জরুরি",
      "বকেয়া",
      "অতিরিক্ত বার্নার",
      "অবৈধ কার্যকলাপ",
      "সাময়িক",
    ],
  },
  afterDisconnection: {
    label: "বিচ্ছিন্নকরণের পর",
    type: "multiselect",
    options: [
      "এই রাইজারে বাণিজ্যিক গ্রাহক থাকিবে",
      "এই রাইজারে আবাসিক গ্রাহক থাকিবে",
      "এই রাইজারে আর কোনও গ্রাহক থাকিবে না",
    ],
  },
  remarks: { label: "মন্তব্য", type: "textarea" },
};

function goBack() {
  emit("close");
}

function formatOptionsWithTicks(fieldKey, selectedValues) {
  const field = fields[fieldKey];
  if (!field || !field.options) return "";
  
  return field.options.map(option => {
    const isSelected = Array.isArray(selectedValues) 
      ? selectedValues.includes(option)
      : selectedValues === option;
    return `${isSelected ? '✓' : '☐'} ${option}`;
  }).join(', ');
}

function generateDocx() {
  const doc = new Document({
    sections: [
      {
        properties: {},
        children: [
          new Paragraph({
            alignment: AlignmentType.CENTER,
            children: [
              new TextRun({
                text: "বাখরাবাদ গ্যাস ডিস্ট্রিবিউশন কোম্পানী লিমিটেড",
                bold: true,
                size: 32,
              }),
            ],
            spacing: { after: 200 },
          }),
          new Paragraph({
            alignment: AlignmentType.CENTER,
            children: [
              new TextRun({
                text: "ডিসকানেকশন রিপোর্ট",
                bold: true,
                size: 28,
              }),
            ],
            spacing: { after: 300 },
          }),
          
          // Header info (Date and Team No)
          new Paragraph({
            alignment: AlignmentType.RIGHT,
            children: [
              new TextRun({ 
                text: `তারিখ: ${headerInfo.value.date || 'N/A'}`, 
                bold: true,
                size: 24
              }),
              new TextRun({ 
                text: `\nটিম নং: ${headerInfo.value.teamNo || 'N/A'}`, 
                bold: true,
                size: 24
              }),
            ],
            spacing: { after: 400 },
          }),
          
          ...Object.entries(fields).map(([key, field]) => {
            const value = entryLocal.value[key];
            let displayValue = "";

            if (key === 'location' && value) {
              displayValue = `অক্ষাংশ: ${value.latitude || 'N/A'}, দ্রাঘিমাংশ: ${value.longitude || 'N/A'}`;
            } else if (field.type === 'select' && field.options) {
              displayValue = formatOptionsWithTicks(key, value);
            } else if (field.type === 'multiselect' && field.options) {
              displayValue = formatOptionsWithTicks(key, value);
            } else if (Array.isArray(value)) {
              displayValue = value.join(", ");
            } else {
              displayValue = value ?? "";
            }

            return new Paragraph({
              children: [
                new TextRun({ 
                  text: `${field.label}: `, 
                  bold: true,
                  size: 24
                }),
                new TextRun({ 
                  text: displayValue,
                  size: 22
                }),
              ],
              spacing: { after: 200 },
            });
          }),
          
          new Paragraph({ 
            text: " ",
            spacing: { after: 400 }
          }),
          
          new Paragraph({
            children: [
              new TextRun({
                text: "গ্রাহক/প্রতিনিধির স্বাক্ষর: ___________________",
                size: 24,
                break: 1,
              }),
            ],
            spacing: { after: 300 },
          }),
          
          new Paragraph({
            children: [
              new TextRun({
                text: "বিজিডিসিএল প্রতিনিধির স্বাক্ষর: ___________________",
                size: 24,
              }),
            ],
            spacing: { after: 300 },
          }),
        ],
      },
    ],
  });

  Packer.toBlob(doc).then((blob) => {
    saveAs(blob, `disconnection_report_${Date.now()}.docx`);
  });
}
</script>

<style scoped>
.report-issue-container {
  padding: 20px;
}

.action-buttons {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

.issue-form {
  margin-top: 20px;
}

.form-label {
  font-weight: 600;
  font-size: 14px;
  margin-bottom: 5px;
  display: block;
}

.form-label-sm {
  font-weight: 500;
  font-size: 12px;
  margin-bottom: 3px;
  display: block;
}

.location-inputs {
  margin-bottom: 10px;
}

.header-info {
  border-top: 2px solid #dee2e6;
  padding-top: 15px;
}

.mb-2 {
  margin-bottom: 15px;
}

.form-control, .form-select {
  font-size: 14px;
}

.btn {
  font-size: 14px;
  padding: 8px 16px;
}

h4 {
  font-size: 18px;
  margin-bottom: 20px;
  color: #2c3e50;
}
</style>