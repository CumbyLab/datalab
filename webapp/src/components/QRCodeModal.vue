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
      // Create print window
      const printWindow = window.open("", "", "height=480, width=1400");
      const doc = printWindow.document;
      // Build DOM using DOM methods
      const html = doc.createElement('html');
      const head = doc.createElement('head');
      const title = doc.createElement('title');
      title.textContent = 'QR Code';
      head.appendChild(title);

      // Inject print CSS directly for maximum reliability
      const printStyle = doc.createElement('style');
      printStyle.type = 'text/css';
      printStyle.textContent = `
@media print {
  @page {
    size: 70mm 24mm;
    padding: 3mm;
  }
  html, body {
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media {
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media.qrcode-modal-flex-row {
    display: grid !important;
    grid-template-columns: 30% 70%;
    grid-template-rows: 100%;
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 100% !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media .qrcode-modal-left {
    width: 100% !important;
    min-width: 0 !important;
    max-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 100% !important;
    box-sizing: border-box !important;
    grid-column: 1;
    grid-row: 1;
  }
  .label-print-media .qrcode-modal-right {
    width: 100% !important;
    min-width: 0 !important;
    max-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 0 !important;
    box-sizing: border-box !important;
    grid-column: 2;
    grid-row: 1;
    margin-left: 0 !important;
    align-items: center;
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
    transform: scale(-1.0, 1.0);
    width: min(70vh, 70vw) !important;
    height: min(70vh, 70vw) !important;
    display: flex !important;
    align-items: center !important;
    justify-content: center !important;
    position: relative !important;
  }
  .label-print-media .qrcode-image > svg,
  .label-print-media .qrcode-image > img {
    width: 100% !important;
    height: 100% !important;
    min-width: 0 !important;
    min-height: 0 !important;
    max-width: 100% !important;
    max-height: 100% !important;
    box-sizing: border-box !important;
    display: block !important;
    padding: 0 !important;
    margin: 0 !important;
  }
  .label-print-media .qrcode-text-label {
    font-size: 14vh !important;
    font-weight: 400;
  }
  .label-print-media .qrcode-sample-name-label {
    font-size: 24vh;
    font-weight: 800;
  }
  .label-print-media .qrcode-sample-chemform-label {
    font-size: 18vh !important;
    font-family: monospace;
  }
}
      `;
      head.appendChild(printStyle);

      html.appendChild(head);
      const body = doc.createElement('body');
      // Build label content
      const wrapper = doc.createElement('div');
      wrapper.className = 'label-print-media qrcode-modal-flex-row';
      // Left (QR)
      const left = doc.createElement('div');
      left.className = 'qrcode-modal-left';
      left.innerHTML = qrHtml;
      // Right (labels)
      const right = doc.createElement('div');
      right.className = 'qrcode-modal-right';
      const nameDiv = doc.createElement('div');
      nameDiv.className = 'qrcode-sample-name-label';
      nameDiv.textContent = name;
      const chemDiv = doc.createElement('div');
      chemDiv.className = 'qrcode-sample-chemform-label';
      chemDiv.textContent = chemform;
      right.appendChild(nameDiv);
      right.appendChild(chemDiv);
      wrapper.appendChild(left);
      wrapper.appendChild(right);
      body.appendChild(wrapper);
      html.appendChild(body);
      doc.replaceChild(html, doc.documentElement);
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
    padding: 0mm;
  }
  html, body {
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media {
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    margin: 0 !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media.qrcode-modal-flex-row {
    display: grid !important;
    grid-template-columns: 30% 70%;
    grid-template-rows: 100%;
    width: 100% !important;
    max-width: 100% !important;
    min-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 100% !important;
    padding: 0 !important;
    box-sizing: border-box !important;
  }
  .label-print-media .qrcode-modal-left {
    width: 100% !important;
    min-width: 0 !important;
    max-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 100% !important;
    box-sizing: border-box !important;
    grid-column: 1;
    grid-row: 1;
    justify-content: right center;
  }
  .label-print-media .qrcode-modal-right {
    width: 100% !important;
    min-width: 0 !important;
    max-width: 100% !important;
    height: 100% !important;
    max-height: 100% !important;
    min-height: 0 !important;
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
  /* Size of the QR image */
  .label-print-media .qrcode-image img {
    transform: scale(-1.0, 1.0);
    width: min(60vh, 100%) !important;
    height: 60vh !important;
    display: flex !important;
    align-items: center !important;
    justify-content: center !important;
    position: relative !important;
  }
  .label-print-media .qrcode-image > svg,
  .label-print-media .qrcode-image > img {
    width: 100% !important;
    height: 100% !important;
    min-width: 0 !important;
    min-height: 0 !important;
    max-width: 100% !important;
    max-height: 100% !important;
    box-sizing: border-box !important;
    display: block !important;
    padding: 0 !important;
    margin: 0 !important;
  }
  .label-print-media .qrcode-text-label {
    min-width: 2mm !important;
    min-height: 2mm !important;
    font-size: clamp(2mm, 8vh, 10mm) !important;
    font-weight: 400
  }
  .label-print-media .qrcode-sample-name-label {
    font-size: 18vh; /* clamp(2mm, 25vh, 8mm) !important; */
    font-weight: 800;
  }
  .label-print-media .qrcode-sample-chemform-label {
    font-size: 12vh !important; /*clamp(2mm, 10vh, 6mm) !important;*/
    font-family: monospace;
  }
  }
</style>
