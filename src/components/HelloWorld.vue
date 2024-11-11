<script setup lang="ts">
import { ref, watch } from 'vue'

defineProps<{
  msg: string
}>()

const emit = defineEmits(['isPlaying'])

class Doot {
  frequency: number
  volume: number
  duration: number
  raw: string
  constructor(note: string) {
    // console.log(`Parsing note: ${note}`)
    this.raw = note
    this.volume = volume.value

    if (note.match(/-/)) {
      this.volume = 0
      this.frequency = 0
    } else {
      this.frequency = parseNoteName(note)
    }

    this.duration = 1 + 0.35 /*The Sushi Constant™️*/ * (note.split('_').length - 1)
    // console.log(`Duration: ${this.duration}`)
  }
}

class AudioContextWrapper {
  ctx: AudioContext
  osc: OscillatorNode
  gain: GainNode
  constructor() {
    this.ctx = new AudioContext()
    this.osc = this.ctx.createOscillator()
    this.gain = this.ctx.createGain()
  }
}

function parseSong(song: string): Array<Doot> {
  const result = []

  for (let i = 0, prev = 0; i < song.length; i++) {
    // Validate character
    if (!song.charAt(i).match(/[A-Z]|[0-9]|-|_|#/)) {
      throw new Error(`Invald character ${song.charAt(i)} at index ${i}`)
    }
    // Split up notes by letter names. (A rest, "-", is a note.)
    else if (i > 0 && song.charAt(i).match(/[A-Z]|-/)) {
      result.push(new Doot(song.slice(prev, i)))
      prev = i
    }
    // If we're at the end, parse whatever we have as a note
    if (i == song.length - 1) {
      result.push(new Doot(song.slice(prev, i + 1)))
      prev = i
    }
  }
  return result
}

// window.AudioContext = window.AudioContext || window.webkitAudioContext // ???
let ctx: AudioContextWrapper
const inputText = ref('')
const volume = ref(0.05) // 5% volume, but still really loud!!!
watch(volume, (nv) => {
  if (ctx) {
    ctx.gain.gain.setValueAtTime(nv, ctx.ctx.currentTime)
  }
})
const isPlaying = ref(false)
watch(isPlaying, (nv) => {
  emit('isPlaying', nv)
})
const defaultText = 'D3D3D4_-A3_--G#3_-G3_-F3_-D3F3G3_-'

function parseNoteName(name: string) {
  const midiNum = nameToMidiNumber(name)

  // Convert MIDI number to frequency (Hz)
  const freq = Math.pow(2, (midiNum - 69) / 12) * 440
  // console.log(`midiNum: ${midiNum}, freq: ${freq}Hz`)

  return freq
}

function nameToMidiNumber(name: string) {
  // Convert it to a MIDI number. A4 is 69.
  // https://newt.phys.unsw.edu.au/jw/notes.html

  let letter: string
  let octave: number
  if (name.charAt(1) == '#') {
    letter = name.slice(0, 2)
    octave = parseInt(name.charAt(2))
  } else {
    letter = name.charAt(0)
    octave = parseInt(name.charAt(1))
  }

  if (!octave) {
    throw new Error('Invalid octave')
  }

  const map = new Map([
    ['C', 0],
    ['C#', 1],
    ['D', 2],
    ['D#', 3],
    ['E', 4],
    ['F', 5],
    ['F#', 6],
    ['G', 7],
    ['G#', 8],
    ['A', 9],
    ['A#', 10],
    ['B', 11],
  ])

  const offset = map.get(letter)
  if (offset === undefined) {
    throw new Error(`Invalid note name: ${letter}`)
  }

  const result = 12 + offset + octave * 12
  // console.log(`Converted "${name}" to MIDI value: ${result}`)
  return result
}

async function onStartButton() {
  if (!inputText.value) {
    inputText.value = defaultText
  }

  try {
    const audioData = parseSong(inputText.value)
    console.log(audioData.map((doot) => doot.raw))

    ctx = ctx || new AudioContextWrapper()

    isPlaying.value = true

    for (const note of audioData) {
      if (!isPlaying.value) {
        return
      }
      playDoot(ctx, note)
      // please don't look at this
      await new Promise((r) => setTimeout(r, 200))
    }
  } catch (e) {
    console.log(e)
  }

  isPlaying.value = false
}

async function onStopButton() {
  isPlaying.value = false
}

function playDoot(ctx: AudioContextWrapper, doot: Doot) {
  const osc = ctx.ctx.createOscillator()
  const gain = ctx.gain

  osc.type = 'sawtooth'
  osc.frequency.value = doot.frequency

  gain.gain.value = Math.min(doot.volume, 0.1)

  osc.connect(gain)
  gain.connect(ctx.ctx.destination)
  osc.start(ctx.ctx.currentTime)
  osc.stop(ctx.ctx.currentTime + 0.175 * doot.duration)
}
</script>

<template>
  <input style="width: 80%" v-model="inputText" placeholder="Type something..." />
  <button v-if="!isPlaying" @click="onStartButton">Play Audio</button>
  <button v-if="isPlaying" @click="onStopButton">Stop Audio</button>
  <input type="range" min="0.0" max="0.1" step="0.01" v-model="volume" />
  <div>{{ volume * 1000 }}%</div>
</template>

<style scoped></style>
