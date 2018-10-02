<template>
  <q-modal
    v-model="opened"
    minimized
    no-backdrop-dismiss
    no-esc-dismiss
    :content-css="{minWidth: '80vw'}"
  >
    <q-modal-layout class="q-pa-xl bg-tertiary text-light">
      <q-btn round class="float-right q-ma-sm" color="info" outline icon="clear" size="md" @click="opened = false">
        <q-tooltip>Return</q-tooltip>
      </q-btn>
      <h4 class="text-info">{{id}}</h4>
      <q-list
        inset-seperator
      >
        <q-item
          v-for="(item, idx) in Object.entries(data)"
          :key="idx"
        >
          <q-item-main>
            <q-item-tile class="text-info">{{item[0]}}: </q-item-tile>
            <q-item-tile>{{item[1]}}</q-item-tile>
          </q-item-main>
        </q-item>
      </q-list>
    </q-modal-layout>
  </q-modal>
</template>

<script>
  export default {
    name: 'State Inspector',
    props: [
      'opened',
      'data',
      'id'
    ]
  }
</script>

<style>
</style>
