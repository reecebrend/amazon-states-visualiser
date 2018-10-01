<template>
  <q-page class="bg-dark text-light">
      <q-layout-header>
        <q-toolbar color="dark">
          <q-toolbar-title>
            Amazon state diagram visualiser
          </q-toolbar-title>
          <q-btn-dropdown label="Examples" class="q-mr-sm" outline text-color="white">
            <q-list link>
              <q-item
                v-for="sm in stateMachines"
                :key="sm.value"
                v-close-overlay
                @click.native="selectStateMachine(sm.shorthand)"
              >
                <q-item-main>
                  <q-item-tile label>{{sm.label}}</q-item-tile>
                </q-item-main>
              </q-item>
            </q-list>
          </q-btn-dropdown>
        </q-toolbar>
      </q-layout-header>
      <div class="row" style="max-height: calc(100vh - 50px); min-height: calc(100vh - 50px);">
        <div class="col-6 col-md-6 shadow-6">
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
        <div class="relative-position col-6">
          <div v-show="displaying === false" class="col-6 q-pa-xl">
            <p class="q-display-3 text-weight-thin">Select a state machine example to visualise!</p>
          </div>

          <div
            id="diagram"
            v-show="displaying === true"
            class="col-6 q-pa-xl scroll text-center"
            style="height: 90%; width: 100%;"
          ></div>
          <p class="q-caption text-light fixed-bottom-right q-pr-md">
            Inspired by the
            <a class="text-info" href="https://github.com/wmfs/tymly-monorepo">Tymly</a>
            implementation of
            <a class="text-info" href="https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html">Amazon States Language</a>
          </p>
        </div>
      </div>

      <div style="position: fixed; bottom: 18px; right: 18px; text-align: right;">
        <q-btn round color="info" outline icon="clear" size="md" style="bottom: 20px; margin-right: 10px;" @click="clear">
          <q-tooltip>Clear</q-tooltip>
        </q-btn>
        <q-btn round color="info" icon="refresh" size="md" style="bottom: 20px" @click="refresh">
          <q-tooltip>Refresh</q-tooltip>
        </q-btn>
      </div>
  </q-page>
</template>

<style>
::-webkit-scrollbar {
  width: 10px;
}
::-webkit-scrollbar-track {
  background: #272822;
}
::-webkit-scrollbar-thumb {
  background: #888;
}
::-webkit-scrollbar-thumb:hover {
  background: #555;
}
</style>

<script>
import fsaStateMachine from '../assets/state-machines/fsa.json'
import simpleTaskMachine from '../assets/state-machines/simple-task-machine.json'
import welcome from '../assets/state-machines/help.json'
import * as opts from '../assets/graph-opts.json'
import Brace from 'vue-bulma-brace'
import * as brace from 'brace'
import * as flowchart from 'flowchart.js'

export default {
  name: 'PageIndex',
  components: {
    Brace
  },
  data: function () {
    return {
      stateCode: '',
      displaying: false,
      stateMachines: {
        fsa: {
          shorthand: 'fsa', label: 'Food Standards Agency', value: JSON.stringify(fsaStateMachine, null, 2)
        },
        stm: {
          shorthand: 'stm', label: 'Simple Task Machine', value: JSON.stringify(simpleTaskMachine, null, 2)
        }
      }
    }
  },
  methods: {
    selectStateMachine (id) {
      console.log('getting: ', id)
      console.log('from: ', this.stateMachines)
      this.stateCode = this.stateMachines[id].value
      this.refresh()
    },
    codeChange (e) {
      this.stateCode = e
    },
    parseStateCode (stateCode) {
      document.getElementById('diagram').innerHTML = ''
      let flowCode = 'st=>start: Start \n'
      let endString = 'st'
      let opCounter = 1
      const states = JSON.parse(stateCode).States
      Object.keys(states).forEach(state => {
        if (states[state].Type === 'Task') {
          flowCode += `op${opCounter}=>operation: ${state} \n`
          endString += `->op${opCounter}`
          opCounter++
        }
      })
      flowCode += `e=>end: End\n${endString}->e`
      const diagram = flowchart.parse(flowCode)
      diagram.drawSVG('diagram', opts.default)
    },
    refresh () {
      this.refreshEditor()
      this.parseStateCode(this.stateCode)
      this.displaying = true
    },
    clear () {
      this.stateCode = ''
      this.displaying = false
      this.refreshEditor()
    },
    refreshEditor () {
      this.editor.session.setValue(this.stateCode)
    }
  },
  mounted () {
    this.editor = brace.edit('vue-bulma-editor')
    this.stateCode = JSON.stringify(welcome, null, 2)
    this.refreshEditor()
  }
}
</script>
