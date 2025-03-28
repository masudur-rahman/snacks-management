<script setup lang="ts">
import { useCurrentUser, useFirestore } from 'vuefire'
import { storeToRefs } from 'pinia'
import { doc, setDoc } from 'firebase/firestore'
import { ref } from 'vue'
import AdminTable from './AdminTable.vue'
import AdminAction from './AdminAction.vue'

import ConfirmModal from './ConfirmModal.vue'
import { type Order, type User, useSnacksStore } from '@/stores/counter'

const snacksStore = useSnacksStore()

await snacksStore.getSnacksEnableUser()
await snacksStore.getAllUser()

const showModal = ref(false)
const isLoading = ref(false)

const user = useCurrentUser()
const { snacksEnabledUsers } = storeToRefs(snacksStore)

function onOnlyPrintClick() {
  window.print()
}

function orderAmountOfUser(orders: Array<Order>) {
  let sum = 0
  orders.forEach(order => sum += (order.amount * order.cost))
  return sum
}

async function updateUserBalance(user: User) {
  const orders = user.orders || []

  const db = useFirestore()

  const url = `/snacks-users/${user?.id}`

  const docRef = doc(db, url)

  const orderAmount = orderAmountOfUser(orders)
  const payload = {
    name: user.name,
    balance: user.balance - orderAmount,
    lastOrdered: new Date().toISOString(),
  }
  await setDoc(docRef, payload, { merge: true })
}

async function onOrderAndPrintClick() {
  try {
    showModal.value = false
    isLoading.value = true
    await Promise.all(
      snacksEnabledUsers.value.map(async (user) => {
        await updateUserBalance(user)
      }),
    )

    await snacksStore.getSnacksEnableUser()
    await snacksStore.getAllUser()

    window.print()
  }
  catch (error) {
    console.error(error)
  }
  isLoading.value = false
}
</script>

<template>
  <div>
    <div class="print:hidden">
      <AdminAction />
    </div>

    <div class="mb-10 flex justify-center">
      <AdminTable :floor="1" />
      <AdminTable :floor="5" />
    </div>
    <div class="flex justify-center print:hidden">
      <button :disabled="isLoading" class="btn btn-primary m-3" @click="onOnlyPrintClick">
        Only Print
      </button>
      <button class="btn btn-primary bg-red-700 m-3" :class="{ loading: isLoading }" @click="showModal = !showModal">
        Order & Print
      </button>
      <router-link to="/balance">
        <button :disabled="isLoading" class="btn btn-primary m-3">
          Print Balance Sheet
        </button>
      </router-link>
    </div>
    <ConfirmModal
      :show-modal="showModal"
      :is-loading="isLoading"
      @close-modal="showModal = false"
      @confirm-modal="onOrderAndPrintClick"
    />
    <p class="hidden print:block">
      Ordered by {{ user?.displayName || user?.email }}
    </p>
  </div>
</template>
