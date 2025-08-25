<template>
  <form class="modal-enclosure" data-testid="qrcode-form">
    <Modal :model-value="modelValue" @update:model-value="$emit('update:modelValue', $event)">
      <template #header>QR Code</template>
      <template #body>
        <div class="form-row">
            <div class="qrcode-modal-flex-row d-flex align-items-center" style="gap: 1.5rem; min-height: 120px;">
              <div ref="qrcode" class="qrcode-modal-left text-center" data-testid="qrcode">
                  <QRCode :refcode="refcode" />
              </div>
              <div class="qrcode-modal-right flex-grow-1">
                <div
                  class="qrcode-sample-name-label mb-2"
                  data-testid="qrcode-sample-name-label"
                  :style="nameFontSizeStyle"
                >
                  {{ name }}
                </div>
                <div class="qrcode-sample-chemform-label" data-testid="qrcode-sample-chemform-label">
                  {{ chemform }}
                </div>
              </div>
            </div>
        </div>
      </template>
      <template #footer>
        <button type="submit" class="btn btn-info" value="Print" @click="printQR">Print</button>
        <button
          type="button"
          class="btn btn-secondary"
          data-dismiss="modal"
          @click="$emit('update:modelValue', false)"
        >
          Close
        </button>
      </template>
    </Modal>
  </form>
</template>

<script>
import Modal from "@/components/Modal.vue";
import QRCode from "@/components/QRCode.vue";
export default {
  name: "QRCodeModal",
  components: {
    QRCode,
    Modal,
  },
  props: {
    modelValue: Boolean,
    refcode: { type: String, required: true },
    name: { type: String, default: '' },
    chemform: { type: String, default: '' },
  },

  emits: ["update:modelValue"],
  computed: {
    nameFontSizeStyle() {
      // Aggressively adjust font size based on name length
      const base = 3.2; // rem
      const min = 0.1; // rem (smaller minimum)
      const maxLen = 15; // start shrinking after this many chars
      if (!this.name) return { fontSize: base + 'rem', fontWeight: 800 };
      let len = this.name.length;
      let size = base;
      if (len > maxLen) {
        size = Math.max(min, base - (len - maxLen) * 0.05); // shrink faster
      }
      return { fontSize: size + 'rem', fontWeight: 800 };
    },
  },
  methods: {
    printQR() {
      const qrHtml = this.$refs.qrcode.innerHTML;
      const name = this.name || '';
      const chemform = this.chemform || '';
      const html = `
        <div class="label-print-media qrcode-modal-flex-row">
          <div class="qrcode-modal-left">
            ${qrHtml}
          </div>
          <div class="qrcode-modal-right">
            <div class="qrcode-sample-name-label">${name}</div>
            <div class="qrcode-sample-chemform-label">${chemform}</div>
          </div>
        </div>
      `;
      const printWindow = window.open("", "", "height=480, width=1400");
      printWindow.document.write(`<html><head><title>QR Code</title><link rel='stylesheet' href='' /></head><body>${html}</body></html>`);
      // Copy styles from the current document's <style> and <link rel="stylesheet"> tags
      const head = document.head.cloneNode(true);
      printWindow.document.head.innerHTML = head.innerHTML;
      printWindow.document.close();
      // Add events to close window after print or cancel
      let printHandled = false;
      function closePrintWindow() {
        if (!printHandled) {
          printHandled = true;
          printWindow.close();
        }
      }
      printWindow.addEventListener('afterprint', closePrintWindow);
      printWindow.addEventListener('focus', function onFocus() {
        setTimeout(closePrintWindow, 250);
        printWindow.removeEventListener('focus', onFocus);
      });
      printWindow.print();
    },
  },
};
</script>

<style>
.modal-enclosure >>> .modal-dialog {
  width: 50vw !important;
  max-width: 50vw !important;
  min-width: 350px;
}
/* Flex row for modal and print, ensures QR code is centered vertically and horizontally */
.qrcode-modal-flex-row {
  display: flex;
  align-items: stretch;
  justify-content: flex-start;
  gap: 1.5rem;
  min-height: 300px;
  height: 100%;
}
/* Left column: fixed width, stack children vertically, prevent overlap */
.qrcode-modal-left {
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
  /* width: 200px;
  min-width: 120px;
  max-width: 200px; */
  flex: 0 0 200px;
  height: 100%;
  box-sizing: border-box;
}

/* Ensure QR code and label stack vertically and are centered */
.qrcode-image {
  width: 100% !important;
  display: flex;
  flex-direction: column;
  align-items: center;
  justify-content: center;
}
.qrcode-text-label {
  width: 100% !important;
  text-align: center;
}
/* Right column: use remaining space, prevent overlap */
.qrcode-modal-right {
  flex: 1 1 0;
  min-width: 80px;
  margin-left: 1rem;
  align-self: flex-start;
  display: flex;
  flex-direction: column;
  justify-content: center;
  box-sizing: border-box;
  height: 100%;
}
.qrcode-modal-right {
  flex: 1 1 67%;
  min-width: 80px;
  margin-left: 1rem;
  align-self: flex-start;
}
.qrcode-sample-name-label {
  font-size: 3.2rem;
  font-weight: 800;
}
.qrcode-sample-chemform-label {
  font-size: 2.2rem !important;
  font-family: monospace;
}
/* Shared QR code and label styles */
.qrcode-image {
  width: 90%;
  display: flex;
  align-items: center;
  justify-content: center;
}
.qrcode-text-label {
  font-size: 2.8mm;
  margin-top: 0.5mm;
  text-align: center;
  width: 100%;
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}
/* Print-specific: restrict height to 24mm for label-print-media */
@media print {
  @page {
    size: 70mm 24mm;
    margin: 0;
  }
  html, body {
    width: 70mm !important;
    max-width: 70mm !important;
    min-width: 70mm !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media {
    width: 70mm !important;
    max-width: 70mm !important;
    min-width: 70mm !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media.qrcode-modal-flex-row {
    display: grid !important;
    grid-template-columns: 20mm 44mm;
    grid-template-rows: 18mm;
    width: 64mm !important;
    max-width: 64mm !important;
    min-width: 64mm !important;
    height: 18mm !important;
    max-height: 18mm !important;
    min-height: 18mm !important;
    margin: 3mm !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media .qrcode-modal-left {
    width: 20mm !important;
    min-width: 0 !important;
    max-width: 20mm !important;
    height: 18mm !important;
    max-height: 18mm !important;
    min-height: 18mm !important;
    box-sizing: border-box !important;
    grid-column: 1;
    grid-row: 1;
  }
  .label-print-media .qrcode-modal-right {
    width: 44mm !important;
    min-width: 0 !important;
    max-width: 44mm !important;
    height: 18mm !important;
    max-height: 18mm !important;
    min-height: 18mm !important;
    box-sizing: border-box !important;
    grid-column: 2;
    grid-row: 1;
    margin-left: 0 !important; /* Align with grid column */
  }
  .label-print-media .qrcode-sample-name-label,
  .label-print-media .qrcode-sample-chemform-label {
    width: 100% !important;
    max-width: 100% !important;
    min-width: 0 !important;
    box-sizing: border-box !important;
    display: block !important;
    text-align: left !important;
    text-overflow: ellipsis !important;
    white-space: normal !important;
  }
  .label-print-media .qrcode-image,
  .label-print-media .qrcode-text-label,
  .label-print-media .qrcode-sample-name-label,
  .label-print-media .qrcode-sample-chemform-label {
    max-height: 100% !important;
    min-height: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media .qrcode-image img {
    transform: scale(1.0) !important; /* Adjust this value if you need to shrink/grow */
    transform-origin: center center !important;
    width: 10mm !important;
    height: 10mm !important;
    display: flex !important;
    align-items: center !important;
    justify-content: center !important;
    position: relative !important;
  }

  .label-print-media .qrcode-image {
    width: 10mm !important;
    height: 10mm !important;
    min-width: 10mm !important;
    min-height: 10mm !important;
    max-width: 10mm !important;
    max-height: 10mm !important;
    box-sizing: border-box !important;
    overflow: hidden !important;
    display: flex !important;
    align-items: center !important;
    justify-content: center !important;
    padding: 0 !important;
    margin: 0 !important;
  }
  .label-print-media .qrcode-image > svg,
  .label-print-media .qrcode-image > img {
    width: 10mm !important;
    height: 10mm !important;
    min-width: 10mm !important;
    min-height: 10mm !important;
    max-width: 10mm !important;
    max-height: 10mm !important;
    box-sizing: border-box !important;
    display: block !important;
    padding: 0 !important;
    margin: 0 !important;
  }
  .label-print-media .qrcode-text-label {
    font-size: 2.5mm !important;
  }
  .label-print-media .qrcode-sample-name-label {
    font-size: 4mm !important;
    font-weight: 800;
  }
  .label-print-media .qrcode-sample-chemform-label {
    font-size: 3mm !important;
    font-family: monospace;
  }
}
</style>
