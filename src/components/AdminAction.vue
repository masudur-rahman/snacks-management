<script setup lang="ts">
import { storeToRefs } from 'pinia'
import { computed, ref } from 'vue'
import ToggleUserModal from './ToggleUserModal.vue'
import { useSnacksStore } from '@/stores/counter'
import AddMoneyModal from '@/components/AddMoneyModal.vue'

const snacksStore = useSnacksStore()

const { allUsers } = storeToRefs(snacksStore)

const showModal = ref(false)

const toggleUser = ref(false)

const totalBalance = computed(() => {
  return allUsers.value.reduce((pre, curr) => {
    return pre += curr.balance || 0
  }, 0) || 0
})
</script>

<template>
  <div>
    <div class="flex justify-around mt-4">
      <div class="card w-96 bg-base-200 space-x-4  p-5">
        <div class="flex justify-around">
          <button class="btn btn-primary" @click="showModal = !showModal">
            Add Money
          </button>
          <button class="btn btn-primary" @click="toggleUser = !toggleUser">
            Toggle user
          </button>
        </div>
      </div>
      <div class="card w-96 bg-base-200  p-5 justify-center">
        <div class="flex space-x-5 items-center justify-between">
          <h2 class="card-title">
            Total Amount
          </h2>
          <div class="form-control">
            <p>{{ totalBalance }} Tk</p>
          </div>
        </div>
      </div>
    </div>
    <AddMoneyModal :show-modal="showModal" @close-modal="showModal = false" />
    <ToggleUserModal :show-modal="toggleUser" @close-modal="toggleUser = false" />
    <div class="divider" />
  </div>
</template>
