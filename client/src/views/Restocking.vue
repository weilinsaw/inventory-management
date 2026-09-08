<template>
  <div class="restocking">
    <div class="page-header">
      <h2>{{ t('restocking.title') }}</h2>
      <p>{{ t('restocking.description') }}</p>
    </div>

    <div v-if="loading" class="loading">{{ t('common.loading') }}</div>
    <div v-else-if="error" class="error">{{ error }}</div>
    <div v-else>
      <div v-if="successMessage" class="success-banner">
        {{ successMessage }}
        <button class="dismiss-button" @click="successMessage = null">&times;</button>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.budgetLabel') }}</h3>
        </div>
        <div class="budget-control">
          <input
            v-model.number="budget"
            type="range"
            min="0"
            max="20000"
            step="100"
            class="budget-slider"
          />
          <div class="budget-value">{{ formatCurrency(budget, currentCurrency) }}</div>
        </div>
        <p class="budget-hint">{{ t('restocking.budgetHint') }}</p>
      </div>

      <div class="card">
        <div class="card-header">
          <h3 class="card-title">{{ t('restocking.recommendedItems') }} ({{ recommendedItems.length }})</h3>
        </div>

        <div v-if="recommendedItems.length === 0" class="no-data">
          {{ t('restocking.emptyState') }}
        </div>
        <div v-else class="table-container">
          <table>
            <thead>
              <tr>
                <th>{{ t('restocking.table.sku') }}</th>
                <th>{{ t('restocking.table.name') }}</th>
                <th>{{ t('restocking.table.trend') }}</th>
                <th>{{ t('restocking.table.quantity') }}</th>
                <th>{{ t('restocking.table.unitCost') }}</th>
                <th>{{ t('restocking.table.lineTotal') }}</th>
              </tr>
            </thead>
            <tbody>
              <tr v-for="item in recommendedItems" :key="item.item_sku">
                <td><strong>{{ item.item_sku }}</strong></td>
                <td>{{ item.item_name }}</td>
                <td>
                  <span :class="['badge', item.trend]">{{ t(`trends.${item.trend}`) }}</span>
                </td>
                <td>{{ item.recommendedQty }}</td>
                <td>{{ formatCurrency(item.unit_cost, currentCurrency) }}</td>
                <td><strong>{{ formatCurrency(item.lineTotal, currentCurrency) }}</strong></td>
              </tr>
            </tbody>
          </table>
        </div>

        <div class="summary-line">
          <span>{{ t('restocking.summary.totalCost') }}: <strong>{{ formatCurrency(totalCommittedCost, currentCurrency) }}</strong></span>
          <span>{{ t('restocking.summary.remainingBudget') }}: <strong>{{ formatCurrency(remainingBudgetAfterOrder, currentCurrency) }}</strong></span>
        </div>

        <div v-if="placeOrderError" class="error">{{ placeOrderError }}</div>

        <div class="place-order-row">
          <button
            class="place-order-button"
            :disabled="recommendedItems.length === 0 || placingOrder"
            @click="placeOrder"
          >
            {{ placingOrder ? t('restocking.placingOrder') : t('restocking.placeOrder') }}
          </button>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { ref, computed, onMounted } from 'vue'
import { api } from '../api'
import { useI18n } from '../composables/useI18n'
import { formatCurrency } from '../utils/currency'

export default {
  name: 'Restocking',
  setup() {
    const { t, currentCurrency } = useI18n()

    const loading = ref(true)
    const error = ref(null)
    const forecasts = ref([])

    const budget = ref(0)
    const placingOrder = ref(false)
    const placeOrderError = ref(null)
    const successMessage = ref(null)

    const loadData = async () => {
      loading.value = true
      error.value = null
      try {
        forecasts.value = await api.getDemandForecasts()
      } catch (err) {
        error.value = t('restocking.loadError')
        console.error(err)
      } finally {
        loading.value = false
      }
    }

    // WHY: Greedily allocate the budget starting with the most urgent (increasing-trend)
    // items, since those are most likely to stock out soon. Within that priority order,
    // fully fund an item's demand gap when the budget allows; when it doesn't, spend
    // whatever budget remains on a partial quantity of that item rather than skipping it
    // outright and leaving budget unused - this maximizes how much of the budget actually
    // goes toward restocking instead of sitting idle.
    const recommendedItems = computed(() => {
      const trendPriority = { increasing: 0, stable: 1, decreasing: 2 }
      const sorted = [...forecasts.value].sort((a, b) => {
        return (trendPriority[a.trend] ?? 3) - (trendPriority[b.trend] ?? 3)
      })

      const results = []
      let remainingBudget = budget.value

      for (const item of sorted) {
        const gap = Math.max(item.forecasted_demand - item.current_demand, 0)
        if (gap <= 0 || !item.unit_cost) continue

        let recommendedQty = 0
        if (remainingBudget >= gap * item.unit_cost) {
          recommendedQty = gap
        } else {
          recommendedQty = Math.floor(remainingBudget / item.unit_cost)
        }

        if (recommendedQty <= 0) continue

        const lineTotal = recommendedQty * item.unit_cost
        remainingBudget -= lineTotal

        results.push({
          item_sku: item.item_sku,
          item_name: item.item_name,
          trend: item.trend,
          unit_cost: item.unit_cost,
          recommendedQty,
          lineTotal
        })
      }

      return results
    })

    const totalCommittedCost = computed(() => {
      return recommendedItems.value.reduce((sum, item) => sum + item.lineTotal, 0)
    })

    const remainingBudgetAfterOrder = computed(() => {
      return Math.max(budget.value - totalCommittedCost.value, 0)
    })

    const placeOrder = async () => {
      placingOrder.value = true
      placeOrderError.value = null
      try {
        await api.createPurchaseOrder({
          supplier_name: 'Acme Parts Supply',
          items: recommendedItems.value.map(i => ({
            item_sku: i.item_sku,
            item_name: i.item_name,
            quantity: i.recommendedQty,
            unit_cost: i.unit_cost
          })),
          notes: 'Restocking order'
        })
        successMessage.value = t('restocking.successMessage')
        budget.value = 0
      } catch (err) {
        placeOrderError.value = t('restocking.placeOrderError')
        console.error(err)
      } finally {
        placingOrder.value = false
      }
    }

    onMounted(loadData)

    return {
      t,
      loading,
      error,
      budget,
      recommendedItems,
      totalCommittedCost,
      remainingBudgetAfterOrder,
      placingOrder,
      placeOrderError,
      successMessage,
      placeOrder,
      currentCurrency,
      formatCurrency
    }
  }
}
</script>

<style scoped>
.budget-control {
  display: flex;
  align-items: center;
  gap: 1.5rem;
  padding: 0.5rem 0;
}

.budget-slider {
  flex: 1;
  accent-color: #3b82f6;
}

.budget-value {
  font-size: 1.5rem;
  font-weight: 700;
  color: #0f172a;
  min-width: 120px;
  text-align: right;
}

.budget-hint {
  font-size: 0.813rem;
  color: #64748b;
  margin-top: 0.5rem;
}

.summary-line {
  display: flex;
  justify-content: space-between;
  padding: 1rem 0.75rem 0;
  font-size: 0.938rem;
  color: #334155;
}

.place-order-row {
  display: flex;
  justify-content: flex-end;
  padding: 1rem 0.75rem 0;
}

.place-order-button {
  padding: 0.75rem 1.5rem;
  background: #3b82f6;
  color: white;
  border: none;
  border-radius: 8px;
  font-weight: 600;
  font-size: 0.938rem;
  cursor: pointer;
  transition: all 0.2s ease;
}

.place-order-button:hover:not(:disabled) {
  background: #2563eb;
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(59, 130, 246, 0.3);
}

.place-order-button:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.success-banner {
  display: flex;
  align-items: center;
  justify-content: space-between;
  background: #d1fae5;
  color: #065f46;
  border: 1px solid #a7f3d0;
  padding: 0.875rem 1.25rem;
  border-radius: 8px;
  margin-bottom: 1.25rem;
  font-size: 0.938rem;
  font-weight: 500;
}

.dismiss-button {
  background: none;
  border: none;
  color: #065f46;
  font-size: 1.25rem;
  cursor: pointer;
  line-height: 1;
  padding: 0 0.25rem;
}

.no-data {
  padding: 2rem;
  text-align: center;
  color: #94a3b8;
  font-size: 0.875rem;
}
</style>
