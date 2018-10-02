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
    refresh () {
      try {
        document.getElementById('diagram').innerHTML = ''
        this.refreshEditor()
        parse(this.stateCode)
        this.getSVGElems()
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
        shape.setAttribute('onclick', `console.log("you clicked: ${text[idx].childNodes[0].innerHTML}")`)
      })
    }
  },
  mounted () {
    this.editor = brace.edit('vue-bulma-editor')
    this.stateCode = JSON.stringify(stateMachines['help'], null, 2)
    this.refreshEditor()
  }
}

const parse = stateMachine => {
  const flatten = (sm, states) => {
    if (sm.States) {
      Object.entries(sm.States).forEach(([key, value]) => {
        if (!states[key]) states[key] = value

        if (value.Branches) {
          value.Branches.forEach(branch => {
            flatten(branch, states)
          })
        }
      })
    }
  }

  // Flatten states
  const sm = JSON.parse(stateMachine)
  const states = {}
  flatten(sm, states)

  console.log('STATES', states)

  const steps = [ 'start=>start: Start', 'end=>end: End' ]
  const flow = [ 'start' ]

  const parseState = (key, state) => {
    console.log(key, state)

    if (state.Type === 'Task') {
      flow.push(key)
      steps.push(`${key}=>operation: ${key}`)
    }

    if (state.Next) {
      parseState(state.Next, states[state.Next])
    }
  }

  parseState(sm.StartAt, states[sm.StartAt])
  flow.push('end')
  const flowCode = `${steps.join('\n')}\n${flow.join('->')}`

  console.log('STEPS', steps)
  console.log('FLOW', flow)
  console.log(flowCode)

  const diagram = flowchart.parse(flowCode)
  diagram.drawSVG('diagram', opts.default)
}

/*
parallel e.g.

start=>start
end=>end
Parallel1=>parallel: Parallel1
Parallel2=>parallel: Parallel2
A=>operation: A
B=>operation: B
C=>operation: C
D=>operation: D
E=>operation: E
F=>operation: F
G=>operation: G

start->Parallel1
Parallel1(path1, left)->A->G
Parallel1(path2, bottom)->B->Parallel2
Parallel2(path1, bottom)->C->F->G
Parallel2(path2, right)->D->E->F->G
G->end

             |
         Parallel1
         |       |
         A       B
     (+4 secs)   |
      |      Parallel2
      |      |       |
      |      C       D
      |  (+2 secs)   |
      |      |       E
      |      |       |
      |      ---------
      |          |
      |          F
      |          |
      ------------
            |
            G
*/

/*
choice e.g.

st=>start
e=>end
cond=>condition: Add or Subtract?
op1=>operation: Add
op2=>operation: Subtract

st->cond
cond(yes)->op1->e
cond(no)->op2->e
*/
</script>
