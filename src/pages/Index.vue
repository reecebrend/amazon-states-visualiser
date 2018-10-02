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
                v-for="(sm, key) in stateMachines"
                :key="key"
                v-close-overlay
                @click.native="selectStateMachine(key)"
                v-if="key !== 'help'"
              >
                <q-item-main>
                  <q-item-tile label>{{sm.Comment}}</q-item-tile>
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
            theme="clouds_midnight"
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

  rect {
    cursor: pointer;
  }
</style>

<script>
import * as stateMachines from '../state-machines'
import * as opts from '../assets/graph-opts.json'
import Brace from 'vue-bulma-brace'
import * as brace from 'brace'
import * as flowchart from 'flowchart.js'
import StateInspector from '../components/state-inspector'

export default {
  name: 'PageIndex',
  components: {
    Brace,
    StateInspector
  },
  data: function () {
    return {
      stateCode: '',
      displaying: false,
      stateMachines
    }
  },
  methods: {
    selectStateMachine (key) {
      this.stateCode = JSON.stringify(this.stateMachines[key], null, 2)
      this.refresh()
    },
    inspectState (state) {
      console.log('inspecting: ', state)
    },
    getSVGElems () {
      const shapes = []
      const text = []
      const svg = document.getElementsByTagName('svg')
      svg[0].childNodes.forEach(node => {
        if (node.nodeName === 'rect') shapes.push(node)
        else if (node.nodeName === 'text') text.push(node)
      })

      shapes.forEach((shape, idx) => {
        const stateId = text[idx].childNodes[0].innerHTML
        shape.addEventListener('click', () => {
          this.inspectState(stateId)
        })
      })
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
      this.getSVGElems()
    },
    refresh () {
      try {
        this.refreshEditor()
        this.parseStateCode(this.stateCode)
        this.displaying = true
      } catch (e) {
        console.log('Error:', e)
        this.$q.notify({
          type: 'warning',
          message: 'There was an error.',
          position: 'top'
        })
      }
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
    this.stateCode = JSON.stringify(stateMachines['help'], null, 2)
    this.refreshEditor()
  }
}
</script>
