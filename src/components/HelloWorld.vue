<script setup lang="ts">
import { ref } from 'vue'

defineProps<{
  msg: string
}>()

class Doot {
  frequency: number
  volume: number
  duration: number
  raw: string
  constructor(note: string) {
    // console.log(`Parsing note: ${note}`)
    this.raw = note

    this.volume = 0.05 // 5% volume, but still really loud!!!

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
let ctx: AudioContext
const inputText = ref('')
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

async function onButton() {
  if (!inputText.value) {
    inputText.value = defaultText
  }

  try {
    const audioData = parseSong(inputText.value)
    console.log(audioData.map((doot) => doot.raw))

    ctx = ctx || new AudioContext()
    for (const note of audioData) {
      playDoot(ctx, note)
      // please don't look at this
      await new Promise((r) => setTimeout(r, 200))
    }
  } catch (e) {
    console.log(e)
  }
}

function playDoot(ctx: AudioContext, doot: Doot) {
  const osc = ctx.createOscillator()
  const gain = ctx.createGain()

  osc.type = 'sawtooth'
  osc.frequency.value = doot.frequency

  gain.gain.value = Math.min(doot.volume, 0.1)

  osc.connect(gain)
  gain.connect(ctx.destination)
  osc.start(ctx.currentTime)
  osc.stop(ctx.currentTime + 0.175 * doot.duration)
}
</script>

<template>
  <div class="greetings">
    <h1 class="green">{{ msg }}</h1>
    <h3>
      <ul>
        <li>To play a note, specify a pitch like E4.</li>
        <li>If you want to elongate the note a bit, you add an underscore like E4_.</li>
        <li>To add a small pause you add a dash -. These can stack unlike the underscores.</li>
        <li>
          <i>
            Note: Long notes play for longer but the longer duration is not taken into account
            before the next note. Insert at least one dash to make notes play properly, like E4_-E4
            would play without a break in between the notes.
          </i>
        </li>
      </ul>
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
