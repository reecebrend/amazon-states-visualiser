<template>
  <q-modal
    v-model="opened"
    minimized
    no-backdrop-dismiss
    no-esc-dismiss
    content-classes="bg-dark"
  >
    <div class="q-pa-xl text-light">
      <div class="row">
        <div class="col-11 q-display-1 q-pl-md">{{id}}</div>
        <div class="col-1 text-right">
          <q-btn flat round icon="clear" color="info" @click="close" />
        </div>
      </div>

      <q-list
        no-border
        dark
      >
        <q-item
          v-for="(item, idx) in Object.entries(data)"
          :key="idx"
        >
          <q-item-main>
            <q-item-tile label>
              {{item[0]}}
            </q-item-tile>
            <q-item-tile sublabel>
              {{item[1]}}
            </q-item-tile>
          </q-item-main>
        </q-item>
      </q-list>
    </div>
  </q-modal>
</template>

<script>
  export default {
    name: 'StateInspector',
    props: [
      'opened',
      'data',
      'id'
    ],
    methods: {
      close () { this.$emit('close') }
    }
  }
</script>

<style>
</style>
