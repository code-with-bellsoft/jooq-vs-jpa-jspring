<template>
  <div class="blackhole-mask leading-none select-none">N+1</div>
</template>

<style>
.blackhole-mask {
  font-family: Georgia, "Times New Roman", "Noto Serif", serif;
  font-weight: 900;
  font-size: 15rem;
  letter-spacing: -0.04em;
  background: #000 url('/backhole.gif') center / cover no-repeat;
  -webkit-background-clip: text;
  background-clip: text;
  -webkit-text-fill-color: transparent;
  color: transparent;
  -webkit-text-stroke: 1px rgba(255, 255, 255, 0.45);
  filter: brightness(1.35) contrast(1.2) drop-shadow(0 0 16px rgba(255, 255, 255, 0.18));
}
</style>
