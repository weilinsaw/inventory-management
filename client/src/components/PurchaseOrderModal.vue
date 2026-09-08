<template>
  <Teleport to="body">
    <Transition name="modal">
      <div v-if="isOpen && backlogItem" class="modal-overlay" @click="close">
        <div class="modal-container" @click.stop>
          <div class="modal-header">
            <h3 class="modal-title">
              {{ mode === 'create' ? t('purchaseOrder.createTitle') : t('purchaseOrder.viewTitle') }}
            </h3>
            <button class="close-button" @click="close">
              <svg width="20" height="20" viewBox="0 0 20 20" fill="none">
                <path d="M15 5L5 15M5 5L15 15" stroke="currentColor" stroke-width="2" stroke-linecap="round"/>
              </svg>
            </button>
          </div>

          <div class="modal-body">
            <!-- Create mode -->
            <form v-if="mode === 'create'" class="po-form" @submit.prevent="submitForm">
              <div class="form-group">
                <label class="form-label">{{ t('purchaseOrder.itemName') }}</label>
                <div class="form-static">{{ backlogItem.item_name }} ({{ backlogItem.item_sku }})</div>
              </div>

              <div class="form-group">
                <label class="form-label" for="po-supplier">{{ t('purchaseOrder.supplierName') }} *</label>
                <input
                  id="po-supplier"
                  v-model="form.supplierName"
                  type="text"
                  class="form-input"
                  required
                />
              </div>

              <div class="form-row">
                <div class="form-group">
                  <label class="form-label" for="po-quantity">{{ t('purchaseOrder.quantity') }} *</label>
                  <input
                    id="po-quantity"
                    v-model.number="form.quantity"
                    type="number"
                    min="1"
                    class="form-input"
                    required
                  />
                </div>

                <div class="form-group">
                  <label class="form-label" for="po-unit-cost">{{ t('purchaseOrder.unitCost') }} *</label>
                  <input
                    id="po-unit-cost"
                    v-model.number="form.unitCost"
                    type="number"
                    min="0"
                    step="0.01"
                    class="form-input"
                    required
                  />
                </div>
              </div>

              <div class="form-group">
                <label class="form-label" for="po-notes">{{ t('purchaseOrder.notes') }}</label>
                <textarea
                  id="po-notes"
                  v-model="form.notes"
                  class="form-textarea"
                  rows="3"
                ></textarea>
              </div>

              <div v-if="submitError" class="error">{{ submitError }}</div>
            </form>

            <!-- View mode -->
            <div v-else>
              <div v-if="viewLoading" class="loading">{{ t('common.loading') }}</div>
              <div v-else-if="viewError" class="error">{{ viewError }}</div>
              <div v-else-if="purchaseOrder">
                <div class="info-grid">
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.supplierName') }}</div>
                    <div class="info-value">{{ purchaseOrder.supplier_name }}</div>
                  </div>
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.status') }}</div>
                    <div class="info-value">
                      <span :class="['badge', getStatusClass(purchaseOrder.status)]">{{ purchaseOrder.status }}</span>
                    </div>
                  </div>
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.createdDate') }}</div>
                    <div class="info-value">{{ formatDate(purchaseOrder.created_date) }}</div>
                  </div>
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.expectedDelivery') }}</div>
                    <div class="info-value">{{ formatDate(purchaseOrder.expected_delivery_date) }}</div>
                  </div>
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.leadTime') }}</div>
                    <div class="info-value">{{ purchaseOrder.lead_time_days }} {{ t('purchaseOrder.days') }}</div>
                  </div>
                  <div class="info-item">
                    <div class="info-label">{{ t('purchaseOrder.totalCost') }}</div>
                    <div class="info-value">{{ formatCurrency(purchaseOrder.total_cost, currentCurrency) }}</div>
                  </div>
                </div>

                <div class="items-table-wrapper">
                  <table class="items-table">
                    <thead>
                      <tr>
                        <th>{{ t('purchaseOrder.table.sku') }}</th>
                        <th>{{ t('purchaseOrder.table.itemName') }}</th>
                        <th>{{ t('purchaseOrder.table.quantity') }}</th>
                        <th>{{ t('purchaseOrder.table.unitCost') }}</th>
                      </tr>
                    </thead>
                    <tbody>
                      <tr v-for="line in purchaseOrder.items" :key="line.item_sku">
                        <td>{{ line.item_sku }}</td>
                        <td>{{ line.item_name }}</td>
                        <td>{{ line.quantity }}</td>
                        <td>{{ formatCurrency(line.unit_cost, currentCurrency) }}</td>
                      </tr>
                    </tbody>
                  </table>
                </div>

                <div v-if="purchaseOrder.notes" class="info-item notes-item">
                  <div class="info-label">{{ t('purchaseOrder.notes') }}</div>
                  <div class="info-value">{{ purchaseOrder.notes }}</div>
                </div>
              </div>
            </div>
          </div>

          <div class="modal-footer">
            <button class="btn-secondary" @click="close">{{ t('common.close') }}</button>
            <button
              v-if="mode === 'create'"
              class="btn-primary"
              :disabled="submitting"
              @click="submitForm"
            >
              {{ submitting ? t('purchaseOrder.submitting') : t('purchaseOrder.submit') }}
            </button>
          </div>
        </div>
      </div>
    </Transition>
  </Teleport>
</template>

<script setup>
import { ref, watch } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'
import { formatCurrency } from '../utils/currency'

const { t, currentCurrency, currentLocale } = useI18n()

const props = defineProps({
  isOpen: {
    type: Boolean,
    default: false
  },
  backlogItem: {
    type: Object,
    default: null
  },
  mode: {
    type: String,
    default: 'create'
  }
})

const emit = defineEmits(['close', 'po-created'])

const form = ref({
  supplierName: '',
  quantity: 0,
  unitCost: 0,
  notes: ''
})

const submitting = ref(false)
const submitError = ref(null)

const purchaseOrder = ref(null)
const viewLoading = ref(false)
const viewError = ref(null)

const resetForm = () => {
  form.value = {
    supplierName: '',
    quantity: props.backlogItem ? props.backlogItem.quantity_needed : 0,
    unitCost: 0,
    notes: ''
  }
  submitError.value = null
}

const loadPurchaseOrder = async () => {
  if (!props.backlogItem) return
  viewLoading.value = true
  viewError.value = null
  purchaseOrder.value = null
  try {
    purchaseOrder.value = await api.getPurchaseOrderByBacklogItem(props.backlogItem.id)
  } catch (err) {
    viewError.value = t('purchaseOrder.loadError')
    console.error(err)
  } finally {
    viewLoading.value = false
  }
}

// Reset/load state whenever the modal is opened, since it stays mounted
// between backlog items (props may not change if the same item is reopened).
watch(() => props.isOpen, (open) => {
  if (open) {
    if (props.mode === 'create') {
      resetForm()
    } else {
      loadPurchaseOrder()
    }
  }
})

const close = () => {
  emit('close')
}

const submitForm = async () => {
  if (!props.backlogItem) return
  submitting.value = true
  submitError.value = null
  try {
    const created = await api.createPurchaseOrder({
      backlog_item_id: props.backlogItem.id,
      supplier_name: form.value.supplierName,
      items: [{
        item_sku: props.backlogItem.item_sku,
        item_name: props.backlogItem.item_name,
        quantity: form.value.quantity,
        unit_cost: form.value.unitCost
      }],
      notes: form.value.notes
    })
    emit('po-created', created)
    emit('close')
  } catch (err) {
    submitError.value = t('purchaseOrder.submitError')
    console.error(err)
  } finally {
    submitting.value = false
  }
}

const getStatusClass = (status) => {
  const statusMap = {
    'Pending': 'info',
    'Delivered': 'success',
    'Shipped': 'info',
    'Processing': 'warning',
    'Backordered': 'danger'
  }
  return statusMap[status] || 'info'
}

const formatDate = (dateString) => {
  if (!dateString) return '-'
  const date = new Date(dateString)
  if (isNaN(date.getTime())) return '-'
  const locale = currentLocale.value === 'ja' ? 'ja-JP' : 'en-US'
  return date.toLocaleDateString(locale, { year: 'numeric', month: 'short', day: 'numeric' })
}
</script>

<style scoped>
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
  padding: 1rem;
}

.modal-container {
  background: white;
  border-radius: 12px;
  box-shadow: 0 20px 50px rgba(0, 0, 0, 0.15);
  max-width: 700px;
  width: 100%;
  max-height: 90vh;
  overflow: hidden;
  display: flex;
  flex-direction: column;
}

.modal-header {
  display: flex;
  align-items: center;
  justify-content: space-between;
  padding: 1.5rem;
  border-bottom: 1px solid #e2e8f0;
}

.modal-title {
  font-size: 1.25rem;
  font-weight: 700;
  color: #0f172a;
  letter-spacing: -0.025em;
}

.close-button {
  background: none;
  border: none;
  color: #64748b;
  cursor: pointer;
  padding: 0.5rem;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 6px;
  transition: all 0.15s ease;
}

.close-button:hover {
  background: #f1f5f9;
  color: #0f172a;
}

.modal-body {
  flex: 1;
  overflow-y: auto;
  padding: 2rem;
}

.info-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 1.5rem;
  margin-bottom: 1.5rem;
}

.info-item {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.notes-item {
  margin-top: 1.5rem;
}

.info-label {
  font-size: 0.813rem;
  font-weight: 600;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  color: #64748b;
}

.info-value {
  font-size: 0.938rem;
  color: #0f172a;
  font-weight: 500;
}

.items-table-wrapper {
  overflow-x: auto;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
}

.items-table {
  width: 100%;
  border-collapse: collapse;
}

.items-table th {
  text-align: left;
  padding: 0.5rem 0.75rem;
  font-weight: 600;
  color: #475569;
  font-size: 0.75rem;
  text-transform: uppercase;
  letter-spacing: 0.05em;
  background: #f8fafc;
  border-bottom: 1px solid #e2e8f0;
}

.items-table td {
  padding: 0.5rem 0.75rem;
  border-top: 1px solid #f1f5f9;
  color: #334155;
  font-size: 0.875rem;
}

.po-form {
  display: flex;
  flex-direction: column;
  gap: 1.25rem;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 1.25rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-label {
  font-size: 0.813rem;
  font-weight: 600;
  color: #475569;
}

.form-static {
  font-size: 0.938rem;
  color: #0f172a;
  font-weight: 500;
}

.form-input,
.form-textarea {
  padding: 0.625rem 0.75rem;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-size: 0.938rem;
  color: #0f172a;
  font-family: inherit;
  transition: border-color 0.15s ease;
}

.form-input:focus,
.form-textarea:focus {
  outline: none;
  border-color: #3b82f6;
}

.form-textarea {
  resize: vertical;
}

.modal-footer {
  padding: 1.5rem;
  border-top: 1px solid #e2e8f0;
  display: flex;
  justify-content: flex-end;
  gap: 0.75rem;
}

.btn-secondary {
  padding: 0.625rem 1.25rem;
  background: #f1f5f9;
  border: 1px solid #e2e8f0;
  border-radius: 8px;
  font-weight: 500;
  font-size: 0.875rem;
  color: #334155;
  cursor: pointer;
  transition: all 0.15s ease;
  font-family: inherit;
}

.btn-secondary:hover {
  background: #e2e8f0;
  border-color: #cbd5e1;
}

.btn-primary {
  padding: 0.625rem 1.25rem;
  background: #3b82f6;
  border: 1px solid #3b82f6;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.875rem;
  color: white;
  cursor: pointer;
  transition: all 0.15s ease;
  font-family: inherit;
}

.btn-primary:hover:not(:disabled) {
  background: #2563eb;
  border-color: #2563eb;
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
}

/* Modal transition animations */
.modal-enter-active,
.modal-leave-active {
  transition: opacity 0.2s ease;
}

.modal-enter-from,
.modal-leave-to {
  opacity: 0;
}

.modal-enter-active .modal-container,
.modal-leave-active .modal-container {
  transition: transform 0.2s ease;
}

.modal-enter-from .modal-container,
.modal-leave-to .modal-container {
  transform: scale(0.95);
}
</style>
