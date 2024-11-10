<script setup lang="ts">
defineProps<{
  msg: string
}>()

console.log('Hello, world')
// window.AudioContext = window.AudioContext || window.webkitAudioContext
window.AudioContext = window.AudioContext
let ctx: AudioContext

function parseNoteName(name: string) {
  const midiNum = nameToMidiNumber(name)

  // Convert MIDI number to frequency (Hz)
  const freq = Math.pow(2, (midiNum - 69) / 12) * 440
  console.log(`midiNum: ${midiNum}, freq: ${freq}Hz`)

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

  let offset
  switch (letter) {
    case 'A':
      offset = 9
      break
    case 'A#':
      offset = 10
      break
    case 'B':
      offset = 11
      break
    case 'C':
      offset = 0
      break
    case 'C#':
      offset = 1
      break
    case 'D':
      offset = 2
      break
    case 'D#':
      offset = 3
      break
    case 'E':
      offset = 4
      break
    case 'F':
      offset = 5
      break
    case 'F#':
      offset = 6
      break
    case 'G':
      offset = 7
      break
    case 'G#':
      offset = 8
      break
    default:
      throw new Error(`Invalid note name: ${letter}`)
  }

  const result = 12 + offset + octave * 12
  console.log(`Converted "${name}" to MIDI value: ${result}`)
  return result
}

async function onButton() {
  const audioData = ['C4', 'D4', 'E4', 'F4', 'G4', 'A4', 'B4', 'C5']
  ctx = ctx || new AudioContext()

  for (const note of audioData) {
    console.log(`Parsing note: ${note}`)
    const freq = parseNoteName(note)
    playDoot(ctx, freq)
    // please don't look at this
    await new Promise((r) => setTimeout(r, 200))
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
  osc.start()

  setTimeout(() => {
    osc.stop()
  }, 175)
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
