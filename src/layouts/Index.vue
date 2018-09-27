<template>
  <q-layout view="lHh Lpr lFf">
    <q-layout-header>
      <q-toolbar
        color="primary"
      >
        <q-toolbar-title>
          Amazon state diagram visualiser
        </q-toolbar-title>
        <q-btn-dropdown label="Examples" class="q-mr-sm" outline text-color="white">
          <q-list link>
            <q-item
              v-for="sm in stateMachines"
              :key="sm.value"
              v-close-overlay
              @click.native="selectStateMachine('fsa')"
            >
              <q-item-main>
                <q-item-tile label>{{sm.label}}</q-item-tile>
              </q-item-main>
            </q-item>
          </q-list>
        </q-btn-dropdown>
      </q-toolbar>
    </q-layout-header>
    <q-page-container>
      <div class="row" style="max-height: calc(100vh - 100px); min-height: calc(100vh - 100px);">
      <div class="col-6 col-md-6">
        <brace
          fontsize="12px"
          theme="monokai"
          mode="json"
          codefolding="markbegin"
          softwrap="free"
          selectionstyle="text"
          highlightline
          style="height: 100%; width: 100%"
          @code-change="codeChange"
        />
      </div>
        <div class="col-6 q-display-3 text-weight-thin q-pa-xl">
          Select a state machine example to visualise!
        </div>
      </div>
    </q-page-container>
  </q-layout>
</template>

<script>
import { openURL } from 'quasar'
import fsaStateMachine from '../assets/state-machines/fsa.json'
import welcome from '../assets/state-machines/help.json'
import Brace from 'vue-bulma-brace'
import * as brace from 'brace'

export default {
  name: 'Index',
  components: {
    Brace
  },
  data: function () {
    return {
      stateCode: '',
      stateMachines: {
        fsa: {
          label: 'Food Standards Agency', value: JSON.stringify(fsaStateMachine)
        }
      }
    }
  },
  methods: {
    openURL,
    selectStateMachine (id) {
      this.stateCode = this.stateMachines[id].value
      this.editor.session.setValue(this.stateCode)
    },
    codeChange (e) {
      console.log('e: ', e)
      this.stateCode = e
    }
  },
  mounted () {
    this.editor = brace.edit('vue-bulma-editor')
    this.editor.session.setValue(this.stateCode)
  }
}
</script>

<style>
</style>
