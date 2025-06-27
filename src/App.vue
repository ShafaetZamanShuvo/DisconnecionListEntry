<template>
  <div class="app">
    <div v-if="!showPrintView">
      <div class="app-header">
        <h2><i class="fas fa-clipboard-list me-2"></i>ডাটা এন্ট্রি অ্যাপ</h2>
      </div>

      <div class="form-container">
        <!-- Statistics -->
        <div class="stats-card">
          <h4>
            <i class="fas fa-chart-bar me-2"></i>মোট এন্ট্রি:
            {{ entries.length }}
          </h4>
        </div>

        <!-- Data Entry Form -->
        <div class="form-card" v-if="showForm">
          <h4 class="mb-3">
            <i class="fas fa-plus-circle me-2"></i>
            {{ editIndex !== null ? 'এন্ট্রি এডিট করুন' : 'নতুন এন্ট্রি' }}
          </h4>

          <form @submit.prevent="saveEntry">
            <div class="mb-3">
              <label class="form-label">কাস্টমার কোড *</label>
              <input
                type="text"
                class="form-control"
                v-model="currentEntry.customerCode"
                required
              />
            </div>

            <div class="mb-3">
              <label class="form-label">কাস্টমার নাম *</label>
              <textarea
                class="form-control"
                rows="1"
                v-model="currentEntry.customerName"
                required
              ></textarea>
            </div>

            <div class="mb-3">
              <label class="form-label">কাস্টমার ঠিকানা *</label>
              <textarea
                class="form-control"
                rows="2"
                v-model="currentEntry.customerAddress"
                required
              ></textarea>
            </div>

            <div class="mb-3">
              <label class="form-label">অনুমোদিত বার্নার *</label>
              <input
                type="text"
                class="form-control"
                v-model="currentEntry.approvedBurner"
                required
              />
            </div>

            <div class="mb-3">
              <label class="form-label">বার্নার পাওয়া গেছে *</label>
              <input
                type="text"
                class="form-control"
                v-model="currentEntry.burnerFound"
                required
              />
            </div>

            <div class="mb-3">
              <label class="form-label">বকেয়া</label>
              <input
                type="number"
                step="0.01"
                class="form-control"
                v-model="currentEntry.due"
              />
            </div>

            <div class="mb-3">
              <label class="form-label">মন্তব্য</label>
              <textarea
                class="form-control"
                rows="2"
                v-model="currentEntry.remarks"
              ></textarea>
            </div>

            <div class="mb-3">
              <label class="form-label">অবস্থান *</label>
              <button
                type="button"
                class="btn btn-info w-100 mb-2"
                @click="getCurrentLocation"
              >
                <i class="fas fa-map-marker-alt me-2"></i>বর্তমান অবস্থান নিন
              </button>
              <div v-if="currentEntry.location" class="location-badge">
                <i class="fas fa-check-circle me-1"></i>অবস্থান সংরক্ষিত
                ({{ currentEntry.location.latitude.toFixed(4) }}, {{ currentEntry.location.longitude.toFixed(4) }})
              </div>
              <div v-else class="text-danger small">
                <i class="fas fa-exclamation-triangle me-1"></i>অবস্থান নেওয়া
                বাধ্যতামূলক
              </div>
            </div>

            <div class="d-grid gap-2">
              <button type="submit" class="btn btn-primary">
                <i class="fas fa-save me-2"></i>
                {{ editIndex !== null ? 'আপডেট করুন' : 'সংরক্ষণ করুন' }}
              </button>
              <button
                type="button"
                class="btn btn-secondary"
                @click="cancelEdit"
              >
                <i class="fas fa-times me-2"></i>
                {{ editIndex !== null ? 'বাতিল করুন' : 'রিসেট করুন' }}
              </button>
            </div>
          </form>
        </div>

        <!-- Data List -->
        <div class="form-card" v-if="!showForm">
          <div class="d-flex justify-content-between align-items-center mb-3">
            <h4><i class="fas fa-list me-2"></i>সংরক্ষিত ডাটা</h4>
            <div>
              <button class="btn btn-success me-2" @click="generatePrintableReport">
                <i class="fas fa-print me-2"></i>প্রিন্ট
              </button>
              <!-- Button to clear all entries -->
              <button class="btn btn-danger" @click="clearAllEntries">
                <i class="fas fa-trash-alt me-2"></i>সব মুছুন
              </button>
            </div>
          </div>

          <div class="data-list" v-if="entries.length > 0">
            <div
              v-for="(entry, index) in entries"
              :key="index"
              class="data-item"
            >
              <div class="row">
                <div class="col-12">
                  <strong>কোড:</strong> {{ entry.customerCode }}<br />
                  <strong>নাম:</strong> {{ entry.customerName }}<br />
                  <strong>ঠিকানা:</strong> {{ entry.customerAddress }}<br />
                  <strong>অনুমোদিত বার্নার:</strong> {{ entry.approvedBurner
                  }}<br />
                  <strong>পাওয়া বার্নার:</strong> {{ entry.burnerFound }}<br />
                  <strong>বকেয়া:</strong>
                  {{ entry.due ? "৳" + entry.due : "নেই" }}<br />
                  <strong>মন্তব্য:</strong> {{ entry.remarks || "নেই" }}<br />
                  <strong>সময়:</strong> {{ formatDate(entry.timestamp) }}<br />
                  <div v-if="entry.location" class="mt-2">
                    <span class="location-badge">
                      <i class="fas fa-map-marker-alt me-1"></i>
                      {{ entry.location.latitude.toFixed(7) }},
                      {{ entry.location.longitude.toFixed(7) }}
                    </span>
                  </div>
                  <div class="d-flex justify-content-start mt-2">
                    <button
                      class="btn btn-info btn-sm me-2"
                      @click="editEntry(entry, index)"
                    >
                      <i class="fas fa-edit"></i> এডিট করুন
                    </button>

                    <button
                      class="btn btn-danger btn-sm"
                      @click="deleteEntry(index)"
                    >
                      <i class="fas fa-trash-alt"></i> মুছুন
                    </button>
                  </div>
                </div>
              </div>
            </div>
          </div>

          <div v-else class="text-center text-muted py-4">
            <i class="fas fa-inbox fa-3x mb-3"></i>
            <p>কোন ডাটা নেই</p>
          </div>
        </div>
      </div>

      <!-- Floating Action Button -->
      <div class="floating-action">
        <button
          class="btn btn-primary rounded-circle"
          @click="toggleView"
          style="width: 60px; height: 60px"
        >
          <i
            :class="showForm ? 'fas fa-list' : 'fas fa-plus'"
            class="fa-lg"
          ></i>
        </button>
      </div>
    </div>

    <BGDCLPrintableReport
      v-if="showPrintView"
      :entries="entries"
      @close="showPrintView = false"
    />
  </div>
</template>

<script>
import { Document, Packer, Paragraph, TextRun } from "docx";
import BGDCLPrintableReport from "./components/BGDCLPrintableReport.vue";

export default {
  name: "App",
  components: { BGDCLPrintableReport },
  data() {
    return {
      showForm: true,
      showPrintView: false,
      editIndex: null,
      currentEntry: {
        customerCode: "",
        customerName: "",
        customerAddress: "",
        approvedBurner: "",
        burnerFound: "",
        due: "",
        remarks: "",
        location: null,
        timestamp: null,
      },
      entries: [],
    };
  },

  mounted() {
    this.loadData();
  },

  methods: {
    editEntry(entry, index) {
      // Set edit mode first
      this.editIndex = index;
      
      // Deep clone the entry to avoid reference issues
      this.currentEntry = {
        customerCode: entry.customerCode || "",
        customerName: entry.customerName || "",
        customerAddress: entry.customerAddress || "",
        approvedBurner: entry.approvedBurner || "",
        burnerFound: entry.burnerFound || "",
        due: entry.due || "",
        remarks: entry.remarks || "",
        location: entry.location ? { ...entry.location } : null,
        timestamp: entry.timestamp
      };
      
      // Switch to form view
      this.showForm = true;
    },

    cancelEdit() {
      if (this.editIndex !== null) {
        // If editing, just cancel and go back to list
        this.editIndex = null;
        this.resetForm();
        this.showForm = false;
      } else {
        // If adding new, just reset the form
        this.resetForm();
      }
    },

    clearAllEntries() {
      if (confirm("আপনি কি নিশ্চিত যে সব এন্ট্রি মুছতে চান?")) {
        this.entries = [];
        this.saveData();
        alert("সব এন্ট্রি সফলভাবে মুছে ফেলা হয়েছে!");
      }
    },

    saveEntry() {
      if (!this.currentEntry.location) {
        alert("অবস্থান নেওয়া বাধ্যতামূলক! অনুগ্রহ করে প্রথমে অবস্থান নিন।");
        return;
      }

      // Only update timestamp for new entries
      if (this.editIndex === null) {
        this.currentEntry.timestamp = new Date().toISOString();
      }

      if (this.editIndex !== null) {
        // Update existing entry
        this.entries.splice(this.editIndex, 1, { ...this.currentEntry });
        this.editIndex = null;
        alert("ডাটা সফলভাবে আপডেট হয়েছে!");
        this.showForm = false; // Go back to list after editing
      } else {
        // Add new entry
        this.entries.push({ ...this.currentEntry });
        alert("ডাটা সফলভাবে সংরক্ষিত হয়েছে!");
      }

      this.saveData();
      this.resetForm();
    },

    deleteEntry(index) {
      if (confirm("আপনি কি নিশ্চিত যে এই এন্ট্রি মুছতে চান?")) {
        this.entries.splice(index, 1);
        this.saveData();
        alert("এন্ট্রি সফলভাবে মুছে ফেলা হয়েছে!");
      }
    },

    resetForm() {
      this.currentEntry = {
        customerCode: "",
        customerName: "",
        customerAddress: "",
        approvedBurner: "",
        burnerFound: "",
        due: "",
        remarks: "",
        location: null,
        timestamp: null,
      };
    },

    getCurrentLocation() {
      if ("geolocation" in navigator) {
        navigator.geolocation.getCurrentPosition(
          (position) => {
            this.currentEntry.location = {
              latitude: position.coords.latitude,
              longitude: position.coords.longitude,
            };
            alert("অবস্থান সফলভাবে সংরক্ষিত হয়েছে!");
          },
          (error) => {
            alert("অবস্থান নিতে সমস্যা হয়েছে। অনুগ্রহ করে অনুমতি দিন।");
          }
        );
      } else {
        alert("আপনার ব্রাউজার জিও-লোকেশন সাপোর্ট করে না।");
      }
    },

    toggleView() {
      if (this.showForm && this.editIndex !== null) {
        // If currently editing, ask for confirmation
        if (confirm("আপনি কি এডিট বাতিল করতে চান?")) {
          this.editIndex = null;
          this.resetForm();
          this.showForm = false;
        }
      } else {
        this.showForm = !this.showForm;
        if (this.showForm) {
          // Reset when showing form
          this.editIndex = null;
          this.resetForm();
        }
      }
    },

    saveData() {
      localStorage.setItem("dataEntries", JSON.stringify(this.entries));
    },

    loadData() {
      const savedData = localStorage.getItem("dataEntries");
      if (savedData) {
        this.entries = JSON.parse(savedData);
      }
    },

    generatePrintableReport() {
      if (this.entries.length === 0) {
        alert("প্রিন্ট করার জন্য কোন ডাটা নেই!");
        return;
      }
      this.showPrintView = true;
    },

    async downloadData() {
      if (this.entries.length === 0) {
        alert("ডাউনলোড করার জন্য কোন ডাটা নেই!");
        return;
      }

      // Create a new DOCX document
      const doc = new Document();

      // Add a title
      doc.addSection({
        children: [
          new Paragraph({
            children: [
              new TextRun({
                text: "ডাটা এন্ট্রি রিপোর্ট",
                bold: true,
                size: 36,
              }),
            ],
          }),
          ...this.entries.map(
            (entry, idx) =>
              new Paragraph({
                children: [
                  new TextRun({
                    text: `\nএন্ট্রি #${idx + 1}\n`,
                    bold: true,
                    size: 28,
                  }),
                  new TextRun(
                    `কাস্টমার কোড: ${entry.customerCode}\n` +
                      `কাস্টমার নাম: ${entry.customerName}\n` +
                      `কাস্টমার ঠিকানা: ${entry.customerAddress}\n` +
                      `অনুমোদিত বার্নার: ${entry.approvedBurner}\n` +
                      `পাওয়া বার্নার: ${entry.burnerFound}\n` +
                      `বকেয়া: ${entry.due || "নেই"}\n` +
                      `মন্তব্য: ${entry.remarks || "নেই"}\n` +
                      `অবস্থান: ${
                        entry.location
                          ? `${entry.location.latitude}, ${entry.location.longitude}`
                          : "নেই"
                      }\n` +
                      `সময়: ${this.formatDate(entry.timestamp)}\n`
                  ),
                ],
              })
          ),
        ],
      });

      // Generate the DOCX blob
      const blob = await Packer.toBlob(doc);

      // Download the DOCX
      const link = document.createElement("a");
      const url = URL.createObjectURL(blob);
      link.href = url;
      link.download = `data-entries-${
        new Date().toISOString().split("T")[0]
      }.docx`;
      document.body.appendChild(link);
      link.click();
      document.body.removeChild(link);
    },

    formatDate(timestamp) {
      if (!timestamp) return "";
      return new Date(timestamp).toLocaleString("bn-BD", {
        year: "numeric",
        month: "short",
        day: "numeric",
        hour: "2-digit",
        minute: "2-digit",
      });
    },
  },
};
</script>

<style>
body {
  font-family: "Arial", sans-serif;
  background-color: #f8f9fa;
  padding: 0;
  margin: 0;
}

.app-header {
  background: linear-gradient(135deg, #007bff, #0056b3);
  color: white;
  padding: 1rem;
  text-align: center;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.1);
}

.form-container {
  max-width: 100%;
  margin: 0;
  padding: 1rem;
}

.form-card {
  background: white;
  border-radius: 15px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
  padding: 1.5rem;
  margin-bottom: 1rem;
}

.form-label {
  font-weight: 600;
  color: #495057;
  margin-bottom: 0.5rem;
}

.form-control,
.form-select {
  border-radius: 10px;
  border: 2px solid #e9ecef;
  padding: 0.75rem;
  font-size: 1rem;
  transition: all 0.3s ease;
}

.form-control:focus,
.form-select:focus {
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0, 123, 255, 0.25);
}

.btn-primary {
  background: linear-gradient(135deg, #007bff, #0056b3);
  border: none;
  border-radius: 10px;
  padding: 0.75rem 2rem;
  font-weight: 600;
  font-size: 1.1rem;
  box-shadow: 0 4px 15px rgba(0, 123, 255, 0.3);
  transition: all 0.3s ease;
}

.btn-primary:hover {
  transform: translateY(-2px);
  box-shadow: 0 6px 20px rgba(0, 123, 255, 0.4);
}

.btn-success {
  background: linear-gradient(135deg, #28a745, #1e7e34);
  border: none;
  border-radius: 10px;
  padding: 0.75rem 2rem;
  font-weight: 600;
  box-shadow: 0 4px 15px rgba(40, 167, 69, 0.3);
}

.btn-info {
  background: linear-gradient(135deg, #17a2b8, #117a8b);
  border: none;
  border-radius: 10px;
  padding: 0.75rem 2rem;
  font-weight: 600;
  box-shadow: 0 4px 15px rgba(23, 162, 184, 0.3);
}

.stats-card {
  background: linear-gradient(135deg, #28a745, #20c997);
  color: white;
  border-radius: 15px;
  padding: 1rem;
  text-align: center;
  margin-bottom: 1rem;
}

.data-list {
  max-height: 400px;
  overflow-y: auto;
}

.data-item {
  background: white;
  border-radius: 10px;
  padding: 1rem;
  margin-bottom: 0.5rem;
  border-left: 4px solid #007bff;
  box-shadow: 0 2px 10px rgba(0, 0, 0, 0.05);
}

.location-badge {
  background: #e9ecef;
  color: #495057;
  padding: 0.25rem 0.5rem;
  border-radius: 15px;
  font-size: 0.8rem;
}

.floating-action {
  position: fixed;
  bottom: 20px;
  right: 20px;
  z-index: 1000;
}

@media (max-width: 576px) {
  .form-container {
    padding: 0.5rem;
  }

  .form-card {
    padding: 1rem;
  }
}
</style>