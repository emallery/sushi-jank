<script setup lang="ts">
import { ref } from 'vue'

defineProps<{
  msg: string
}>()

class Doot {
  frequency: number
  volume: number
  duration: number
  constructor(note: string) {
    console.log(`Parsing note: ${note}`)

    this.volume = 0.05 // 5% volume, but still really loud!!!
    this.duration = 1
    this.frequency = parseNoteName(note)
  }
}

function parseSong(song: string): Array<Doot> {
  const result = []

  for (let i = 0, prev = 0; i < song.length; i++) {
    // Validate character
    if (!song.charAt(i).match(/[A-Z]|[0-9]|-|_/)) {
      throw new Error(`Invald character ${song.charAt(i)} at index ${i}`)
    }
    // If we're at the end, parse whatever we have as a note
    else if (i == song.length - 1) {
      result.push(new Doot(song.slice(prev, i + 1)))
      prev = i
    }
    // Otherwise, split up notes by letter names
    else if (i > 0 && song.charAt(i).match(/[A-Z]/)) {
      result.push(new Doot(song.slice(prev, i)))
      prev = i
    }
  }
  return result
}

// window.AudioContext = window.AudioContext || window.webkitAudioContext
let ctx: AudioContext
const inputText = ref('')
const defaultText = 'C4D4E4F4G4A4B4C5'

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

async function onButton() {
  if (!inputText.value) {
    inputText.value = defaultText
  }

  try {
    const audioData = parseSong(inputText.value)

    ctx = ctx || new AudioContext()
    for (const note of audioData) {
      playDoot(ctx, note.frequency)
      // please don't look at this
      await new Promise((r) => setTimeout(r, 200))
    }
  } catch (e) {
    console.log(e)
  }
}

function playDoot(ctx: AudioContext, frequency: number) {
  const osc = ctx.createOscillator()
  const gain = ctx.createGain()

  osc.type = 'sawtooth'
  osc.frequency.value = frequency

  gain.gain.value = 0.04

  osc.connect(gain)
  gain.connect(ctx.destination)
  osc.start(ctx.currentTime)
  osc.stop(ctx.currentTime + 0.175)
}
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    <h3>
      You’ve successfully created a project with
      <a href="https://vite.dev/" target="_blank" rel="noopener">Vite</a> +
      <a href="https://vuejs.org/" target="_blank" rel="noopener">Vue 3</a>. What's next?
    </h3>
  </div>
  <input v-model="inputText" placeholder="Type something..." />
  <button @click="onButton">Play Audio</button>
</template>

<style scoped>
h1 {
  font-weight: 500;
  font-size: 2.6rem;
  position: relative;
  top: -10px;
}

h3 {
  font-size: 1.2rem;
}

.greetings h1,
.greetings h3 {
  text-align: center;
}

@media (min-width: 1024px) {
  .greetings h1,
  .greetings h3 {
    text-align: left;
  }
}
</style>
