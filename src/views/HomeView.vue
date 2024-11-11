<script setup lang="ts"></script>

<template>
  <main>
    <div class="greetings">
      <h1 class="green">How to Use!</h1>
      <h3>
        <ul>
          <li>To play a note, specify a pitch like E4.</li>
          <li>If you want to elongate the note a bit, you add an underscore like E4_.</li>
          <li>To add a small pause you add a dash -. These can stack unlike the underscores.</li>
          <li>
            <i>
              Note: Long notes play for longer but the longer duration is not taken into account
              before the next note. Insert at least one dash to make notes play properly, like
              E4_-E4 would play without a break in between the notes.
            </i>
          </li>
        </ul>
      </h3>
    </div>
  </main>
</template>

<style lang="css" scoped>
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
