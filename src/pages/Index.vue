<template>
  <q-page>
      <q-layout-header>
        <q-toolbar
          color="dark"
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
        <div class="relative-position col-6">
        <div v-show="this.displaying === false" class="col-6 q-pa-xl">
          <p class="q-display-4 text-weight-thin">Select a state machine example to visualise!</p>
          <p class="q-caption text-weight-thin fixed-bottom-right q-pr-md">Inspired by the <a href="https://github.com/wmfs/tymly-monorepo">Tymly</a> implementation of <a href="https://docs.aws.amazon.com/step-functions/latest/dg/concepts-amazon-states-language.html">Amazon States Language</a></p>
        </div>
        <div id="diagram" v-show="this.displaying === true" class="absolute-center col-6 q-pa-xl">
        </div>
      </div>
      </div>
  </q-page>
</template>

<style>
</style>

<script>
  import fsaStateMachine from '../assets/state-machines/fsa.json'
  import simpleStateMachine from '../assets/state-machines/simple-task-machine.json'
  import welcome from '../assets/state-machines/help.json'
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
            shorthand: 'stm', label: 'Simple Task Machine', value: JSON.stringify(simpleStateMachine, null, 2)
          }
        }
      }
    },
    methods: {
      selectStateMachine (id) {
        this.stateCode = this.stateMachines[id].value
        this.editor.session.setValue(this.stateCode)
        this.parseStateCode(this.stateCode)
        this.displaying = true
      },
      codeChange (e) {
        this.stateCode = e
      },
      parseStateCode (stateCode) {
        let flowCode = 'st=>start: Start \n'
        let endString = 'st'
        let opCounter = 1
        const states = JSON.parse(stateCode).States
        Object.keys(states).forEach(state => {
          if (states[state].Type === 'Task') {
            flowCode += `op${opCounter}=>operation: ${state} \n`, endString += `->op${opCounter}`, opCounter++
          }
        })
        flowCode += `e=>end: End\n${endString}->e`
        const diagram = flowchart.parse(flowCode)
        diagram.drawSVG('diagram')
      }
    },
    mounted () {
      this.editor = brace.edit('vue-bulma-editor')
      this.editor.session.setValue(JSON.stringify(welcome, null, 2))
    }
  }
</script>
