<template>
  <form class="modal-enclosure" data-testid="qrcode-form">
    <Modal :model-value="modelValue" @update:model-value="$emit('update:modelValue', $event)">
      <template #header>QR Code</template>
      <template #body>
        <div class="form-row">
          <div class="qrcode-modal-flex-row d-flex align-items-center">
            <div ref="qrcode" class="qrcode-modal-left text-center" data-testid="qrcode">
              <QRCode :refcode="refcode" />
            </div>
            <div class="qrcode-modal-right flex-grow-1">
              <div class="qrcode-sample-name-label mb-2" data-testid="qrcode-sample-name-label">
                {{ modalName }}
              </div>
              <div class="qrcode-sample-chemform-label" data-testid="qrcode-sample-chemform-label">
                {{ modalChemform }}
              </div>
            </div>
          </div>
        </div>
      </template>
      <template #footer>
        <button type="button" class="btn btn-info" value="Print" @click="printQR">Print</button>
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
  },

  emits: ["update:modelValue"],
  computed: {
    itemID() {
      const direct = this.$store.state.refcode_to_id[this.refcode];
      if (direct) {
        return direct;
      }

      const allItemData = this.$store.state.all_item_data || {};
      for (const [itemID, itemData] of Object.entries(allItemData)) {
        if (itemData?.refcode === this.refcode || itemID === this.refcode) {
          return itemID;
        }
      }

      return null;
    },
    itemData() {
      if (!this.itemID) {
        return null;
      }
      return this.$store.state.all_item_data[this.itemID] || null;
    },
    modalName() {
      return this.itemData?.name || "";
    },
    modalChemform() {
      return this.itemData?.chemform || this.itemData?.characteristic_chemical_formula || "";
    },
  },
  methods: {
    printQR() {
      const qrHtml = this.$refs.qrcode.innerHTML;
      const name = this.modalName || '';
      const chemform = this.modalChemform || '';
      // Create print window
      const printWindow = window.open("", "", "height=480, width=1400");
      const doc = printWindow.document;

      const basePath = process.env.BASE_URL || "/";
      const normalizedBasePath = basePath.endsWith("/") ? basePath : `${basePath}/`;
      const overrideCssHref = `${window.location.origin}${normalizedBasePath}custom/override.css`;

      // Build DOM using DOM methods
      const html = doc.createElement('html');
      const head = doc.createElement('head');
      const title = doc.createElement('title');
      title.textContent = 'QR Code';
      head.appendChild(title);

      const overrideLink = doc.createElement('link');
      overrideLink.rel = 'stylesheet';
      overrideLink.href = overrideCssHref;
      head.appendChild(overrideLink);

      html.appendChild(head);
      const body = doc.createElement('body');
      // Build label content
      const wrapper = doc.createElement('div');
      wrapper.className = 'label-print-media qrcode-modal-flex-row';
      // Left (QR)
      const left = doc.createElement('div');
      left.className = 'qrcode-modal-left';
      left.innerHTML = qrHtml;
      left
        .querySelectorAll(
          '.qrcode-status-panel, .shareable-link, .qrcode-generate-public-panel, .qrcode-purl-warning, .alert, button',
        )
        .forEach((element) => element.remove());
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

      const triggerPrint = () => {
        // Small delay helps some label-printer drivers fully apply external CSS.
        setTimeout(() => {
          printWindow.print();
        }, 50);
      };

      overrideLink.onload = triggerPrint;
      overrideLink.onerror = triggerPrint;

      // Fallback in case onload is not fired by the browser/driver.
      setTimeout(triggerPrint, 300);
    },
  },
};
</script>
