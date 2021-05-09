<template>
  <div class="notebook">
    <aside class="side-bar">
      <div class="toolbar">
        <button @click="addNote"
        :title="addButtonTitle">
          <i class="material-icons">+</i> Add note
        </button>
      </div>
      <div class="notes">
        <div class="note" v-for="(note, index) in sortedNotes" :key="index"
        @click="selectNote(note)"
        :class="{selected: note === selectedNote}"
        >
        <i class="icon material-icons" v-if="note.favorite">unstar</i>
        {{ note.title }}</div>
      </div>
    </aside>

    <template v-if="selectedNote">
      <section class="main">
        <div class="toolbar">
          <input v-model="selectedNote.title" placeholder="Note title" />

          <button @click="favoriteNote" title="Favorite note">
            <i class="material-icons">
              {{ selectedNote.favorite ? 'unstar' : 'star' }}
            </i>
          </button>

          <button @click="removeNote" title="Remove note">
            <i class="material-icons">delete</i>
          </button>
        </div>
        <textarea v-model="content">
        </textarea>
        <div class="toolbar status-bar">
          <span class="date">
            <span class="label">Created</span>
            <span class="value">{{ selectedNote.created | date}}</span>
          </span>
          <span class="lines">
            <span class="label">Lines</span>
            <span class="value">{{ linesCount }}</span>
          </span>
          <span class="words">
            <span class="label">Words</span>
            <span class="value">{{ wordsCount }}</span>
          </span>
          <span class="characters">
            <span class="label">Characters</span>
            <span class="value">{{ charactersCount }}</span>
          </span>
        </div>
      </section>
      <aside class="preview" v-html="notePrview">

      </aside>
    </template>
  </div>
</template>

<script>
import marked from 'marked'
export default {
  data () {
    return {
      content: localStorage.getItem('content') || 'You can write in ** markdown **',
      notes: JSON.parse(localStorage.getItem('notes')) || [],
      selectedId: localStorage.getItem('selected-id') || null
    }
  },
  computed: {
    selectedNote () {
      return this.notes.find(note => note.id === this.selectedId)
    },
    notePrview () {
      return marked(this.content)
    },
    addButtonTitle () {
      return this.notes.length + ' note(s) already'
    },
    sortedNotes() {
      return this.notes.slice()
        .sort((a, b) => a.created - b.created)
        .sort((a, b) => (a.favorite === b.favorite) ? 0
          : a.favorite ? -1
          : 1)
    },
    linesCount() {
      let linesCount
      if (this.selectedNote) {
        linesCount = this.selectedNote.content.split(/\r\n|\r|\n/).length
      }
      return linesCount
    },
    wordsCount() {
      let wordsCount
      if (this.selectedNote) {
        let s = this.selectedNote.content
        s = s.replace(/\n/g, ' ')
        s = s.replace(/(^\s*)|(\s*$)/gi, '')
        s = s.replace(/[ ]{2,}/gi, ' ')
        wordsCount =  s.split(' ').length
      }
      return wordsCount
    },
    charactersCount() {
      let charactersCount
      if (this.selectedNote) {
        charactersCount = this.selectedNote.content.split('').length
      }
      return charactersCount
    }
  },
  watch: {
    notes: {
      handler: 'saveNotes',
      deep: true,
    },
    selectedId (val) {
      localStorage.setItem('selected-id', val)
    },
  },
  methods: {
    favoriteNote() {
      this.selectedNote.favorite ^= true
    },
    removeNote() {
      if (this.selectedNote && confirm('Delete the note?')) {
        const index = this.notes.indexOf(this.selectedNote)
        if (index !== -1) {
          this.notes.splice(index, 1)
        }
      }
    },
    selectNote(note) {
      this.selectedId = note.id
    },
    saveNote() {
      localStorage.setItem('content', this.content)
    },
    addNote() {
      const time = Date.now()
      const note = {
        id: String(time),
        title: `New note ${this.notes.length}`,
        content: '**Hi!**This notebook is using [markdown] for formatting!',
        created: time,
        favorite: false
      }
      this.notes.push(note)
      this.selectNote(note)
    },
    saveNotes () {
      localStorage.setItem('notes', JSON.stringify(this.notes))
    },
  }
}
</script>

<style>
body {
  font-family: sans-serif;
  font-size: 16px;
  height: 100%;
  margin: 0;
  box-sizing: border-box;
}

.material-icons {
  font-size: 24px;
  line-height: 1;
  vertical-align: middle;
  margin: -3px;
  padding-bottom: 1px;
}

.notebook > * {
  float: left;
  display: flex;
  flex-direction: column;
  height: 100%;
  flex: auto 0 0;
}

.side-bar {
  background: #f8f8f8;
  width: 20%;
  box-sizing: border-box;
  height: 100vh;
}

.note {
  padding: 16px;
  cursor: pointer;
}

.note:hover {
  background: #ade2ca;
}

.note .icon {
  float: right;
}

button,
input,
textarea {
  font-family: inherit;
  font-size: inherit;
  line-height: inherit;
  box-sizing: border-box;
  outline: none !important;
}

button,
.note.selected {
  background: #40b883;
  color: white;
}

button {
  border-radius: 3px;
  border: none;
  display: inline-block;
  padding: 8px 12px;
  cursor: pointer;
  margin: 0 5px;
}

button:hover {
  background: #63c89b;
}

input {
  border: solid 2px #ade2ca;
  border-radius: 3px;
  padding: 6px 10px;
  background: #f0f9f5;
  color: #666;
}

input:focus {
  border-color: #40b883;
  background: white;
  color: black;
}

button,
input {
  height: 34px;
}

.main, .preview {
  width: 40%;
  height: 100vh;
}

.toolbar {
  padding: 4px;
  box-sizing: border-box;
}

.toolbar span {
  margin: 0 2px;
}

.status-bar {
  color: #999;
  font-style: italic;
}

textarea {
  resize: none;
  border: none;
  box-sizing: border-box;
  margin: 0 4px;
  font-family: monospace;
}

textarea, .notes, .preview {
  flex: auto 1 1;
  overflow: auto;
}

.preview {
  padding: 12px;
  box-sizing: border-box;
  border-left: solid 4px #f8f8f8;
}

.preview p:first-child {
  margin-top: 0;
}

a {
  color: #40b883;
}

h1,
h2,
h3 {
  margin: 10px 0 4px;
  color: #40b883;
}

h1 {
  font-size: 2em;
}

h2 {
  font-size: 1.5em;
}

h3 {
  font-size: 1.2em;
}

h4 {
  font-size: 1.1em;
  font-weight: normal;
}
</style>
