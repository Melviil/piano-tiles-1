<template>
  <div ref="page" class="page__container" @click="play">
    <div class="relative">
      <canvas ref="canvas"></canvas>
      <div class="absolute top-0">
        <audio ref="audio_player" src="/chicago.ogg" />
        {{Math.floor(curr_timestamp)}}
      </div>
      <div class="buttons__container">
        <button ref="key_0" v-touch:start="() => { keyDown(0) }" v-touch:end="() => { keyUp(0) }" />
        <button ref="key_1" v-touch:start="() => { keyDown(1) }" v-touch:end="() => { keyUp(1) }" />
        <button ref="key_2" v-touch:start="() => { keyDown(2) }" v-touch:end="() => { keyUp(2) }" />
        <button ref="key_3" v-touch:start="() => { keyDown(3) }" v-touch:end="() => { keyUp(3) }" />
      </div>
      <div class="row">
      <div class="col "><p>{{this.score_key_0}}</p></div>
      <div class="col "><p>{{this.score_key_1}}</p></div>
      <div class="col "><p>{{this.score_key_2}}</p></div>
      <div class="col "><p>{{this.score_key_3}}</p></div>
      <div class="absolute top-0 col">
        <p>
          </p>
        <!-- {{this.score}} -->
      </div>
    </div>
    </div>
  </div>
</template>

<script >
import sheet from "~/assets/json/chicago.json";
export default {
  data: () => ({
    canvas_height: undefined,
    canvas_width: undefined,
    ctx: undefined,

    score: 0,
    score_key_0: "",
    score_key_1: "",
    score_key_2: "",
    score_key_3: "",

    curr_timestamp: undefined,
    prev_timestamp: undefined,
    start_timestamp: undefined,

    is_playing: false,
    tile_list: [],
    scroll_speed: 3000, // 3000ms for a tile to cross the entire screen
  }),

  mounted() {
    // Fit canvas to page dimensions
    const page = this.$refs.page;
    const canvas = this.$refs.canvas;
    canvas.height = page.clientHeight;
    canvas.width = page.clientWidth;
    this.canvas_height = canvas.height;
    this.canvas_width = canvas.width;
    this.ctx = canvas.getContext("2d");

    // Add keyboard event listener
    document.addEventListener("keydown", (event) => {
      let key_id = event.key == "d" ? 0 : event.key == "f" ? 1 : event.key == "j" ? 2 : event.key == "k" ? 3 : null;
      console.log(`{ "key": ${key_id}, "time": ${this.curr_timestamp - 20} },`);
      console.log('Score', this.score)
      this.score = this.score + this.GetPoints(key_id, this.curr_timestamp - 20);
      console.log('Score', this.score)
    });

    // Pre-draw keys line separators
    this.ctx.beginPath();
    this.ctx.strokeStyle = "#0007";
    this.ctx.moveTo(this.canvas_width * 0.25, 0);
    this.ctx.lineTo(this.canvas_width * 0.25, this.canvas_height);
    this.ctx.moveTo(this.canvas_width * 0.5, 0);
    this.ctx.lineTo(this.canvas_width * 0.5, this.canvas_height);
    this.ctx.moveTo(this.canvas_width * 0.75, 0);
    this.ctx.lineTo(this.canvas_width * 0.75, this.canvas_height);
    this.ctx.stroke();
  },

  methods: {
    draw(timestamp) {
      if (!this.start_timestamp) this.start_timestamp = timestamp;
      this.curr_timestamp = timestamp - this.start_timestamp;
      // Clear canvas
      this.ctx.clearRect(0, 0, this.canvas_width, this.canvas_height);
      // Keys line separators
      this.ctx.beginPath();
      this.ctx.strokeStyle = "#0007";
      this.ctx.moveTo(this.canvas_width * 0.25, 0);
      this.ctx.lineTo(this.canvas_width * 0.25, this.canvas_height);
      this.ctx.moveTo(this.canvas_width * 0.5, 0);
      this.ctx.lineTo(this.canvas_width * 0.5, this.canvas_height);
      this.ctx.moveTo(this.canvas_width * 0.75, 0);
      this.ctx.lineTo(this.canvas_width * 0.75, this.canvas_height);
      this.ctx.stroke();

      // Calculate speed
      let px_per_ms = this.canvas_height / this.scroll_speed;

      // Tiles drawing
      sheet.forEach((tile) => {
        // 480: distance from bottom to button
        let start_position = -tile.time + (this.scroll_speed - 480);
        let x = this.canvas_width * 0.25 * tile.key + this.canvas_width * 0.125;
        // -30: arbitrary delay offset (-30 for mobile / +30 for desktop ??)
        let y = (start_position + this.curr_timestamp) * px_per_ms - 30;
        this.ctx.save();
        this.ctx.beginPath();
        this.ctx.strokeStyle = "black";
        this.ctx.arc(x, y, 22, 0, 2 * Math.PI);
        this.ctx.fill();
        this.ctx.restore();
      });

      this.prev_timestamp = timestamp;
      requestAnimationFrame(this.draw);
    },

    play() {
      if (this.is_playing) return;
      
      this.$refs.audio_player.play();
      requestAnimationFrame(this.draw);
      this.is_playing = true;
    },

    keyDown(key_id) {
      this.$refs[`key_${key_id}`].classList.add("active");  
      // TODO: check tile
    },
    keyUp(key_id) {
      this.$refs[`key_${key_id}`].classList.remove("active");
    },
    displayScore(text, id){
      if (id == 0) this.score_key_0 = text;
      else if (id == 1) this.score_key_1 = text;
      else if (id == 2) this.score_key_2 = text;
      else if (id == 3) this.score_key_3 = text;
      setTimeout(() => {  this.score_key_0 = "" }, 300);
      setTimeout(() => {  this.score_key_1 = "" }, 300);
      setTimeout(() => {  this.score_key_2 = "" }, 300);
      setTimeout(() => {  this.score_key_3 = "" }, 300);

    },
    GetPoints(key_id, timestamp){  
      console.log('enter', timestamp)
      var x = 300
      for (var i = 0; i < sheet.length; i++) {
        if (sheet[i].key == key_id){
          if ( sheet[i].time + x > timestamp && timestamp > sheet[i].time - x){
            this.displayScore("PERFECT", sheet[i].key);
            return 4;
          }
          else if ( sheet[i].time + 350 > timestamp && timestamp > sheet[i].time - 350 ){
            this.displayScore("GOOD", sheet[i].key);
            return 3;
          }
          else if ( sheet[i].time + 400 > timestamp && timestamp > sheet[i].time - 400 ){
            this.displayScore("BAD", sheet[i].key);
            console.log('BAD', sheet[i].key, sheet[i].time, timestamp);
            return 2;
          }
          else if ( sheet[i].time + 450 > timestamp && timestamp > sheet[i].time - 450 ){
            this.displayScore("POOR", sheet[i].key);
            console.log('POOR', sheet[i].key, sheet[i].time, timestamp)
            return 1;
          }
        }
      }

      return 0;


      // print(timestamp - perfect_timestamp)
      // if (perfect_timestamp - timestamp > 4) return Score.Perfect;
      // if (perfect_timestamp - timestamp > 2) return Score.Good;
      // if (perfect_timestamp - timestamp > 2) return Score.Poor;
      // if (perfect_timestamp - timestamp > 2) return Score.Bad;
        
    },
    getValues: function(object) {
      // use a polyfill in case Object.values is not supported by current browser
      return Object.values ? Object.values(object) : Object.keys(object).map(key => object[key]);
    }
  },
};
</script>

<style lang="postcss">
.page__container {
  @apply h-full max-w-2xl;
  @apply mx-auto;
  @apply flex flex-col justify-center;
}
canvas {
  @apply rounded-lg;
  box-shadow: 0 15px 70px rgb(0 0 0 / 10%);
}

.buttons__container {
  @apply absolute;
  bottom: 10%;
  @apply w-full;
  @apply flex justify-around;
}
button {
  @apply h-14 w-14;
  @apply rounded-full;
  @apply transition-all;
  background: linear-gradient(145deg, #e6e6e6, #ffffff);
  box-shadow: 12px 12px 24px #d1d1d1, -12px -12px 24px #ffffff;

  @media (min-width: 1024px) {
    @apply h-20 w-20;
    background: linear-gradient(145deg, #dddddd, #ffffff);
    box-shadow: 18px 18px 36px #bebebe, -18px -18px 36px #ffffff;
  }
}
button.active {
  background: linear-gradient(145deg, #cacaca, #f0f0f0);
  box-shadow: 1px 1px 2px #d1d1d1, -1px 1px 2px #ffffff;

  @media (min-width: 1024px) {
    box-shadow: 5px 5px 10px #bebebe, -5px -5px 10px #ffffff;
  }
}
</style>

<style lang="postcss">
html,
body,
#__nuxt,
#__layout {
  @apply h-full;
}
.row {
  display: flex; /* equal height of the children */
}

.col {
  flex: 1; /* additionally, equal width */
  
  padding: 1em;
  border: solid;
}
</style>