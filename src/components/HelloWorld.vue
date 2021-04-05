<template>
  <div v-if="isGameReady">
    <img class="logo" src="../assets/images/home.svg">
    <p class="title">길토배고</p>
    <button class="btn-confirm" @click="init">START GAME</button>
  </div>
  <div v-else class="maze" @keydown="directionEvent" tabindex="0" ref="game">
    <div v-for='(row, index) in maze' v-bind:key='index' class="maze-con">
      <template v-for='(col, _index) in row'>
        <span v-if="col === 0" v-bind:key="'_' + _index"></span>
        <img v-else-if="col === 1" src="../assets/images/bush.svg" v-bind:key="'_' + _index">
        <img v-else-if="col === 2" src="../assets/images/home.svg" v-bind:key="'_' + _index">
        <img :class="lookingLeft ? 'left' : null" v-else-if="col === 3" src="../assets/images/rabbit.gif" v-bind:key="'_' + _index">
        <img v-else-if="col === 4" src="../assets/images/carrot.svg" v-bind:key="'_' + _index">
      </template>
    </div>
    <br/>
    <div>score : <span>{{score}}</span></div>
    <div>time : <span>{{time}}</span></div>
    <div class="arrow-wrap">
      <button class="btn-arrow top" @click="handlePressUp"><img src="../assets/images/arrow.svg" /></button>
      <button class="btn-arrow right" @click="handlePressRight"><img src="../assets/images/arrow.svg" /></button>
      <button class="btn-arrow bottom" @click="handlePressDown"><img src="../assets/images/arrow.svg" /></button>
      <button class="btn-arrow left" @click="handlePressLeft"><img src="../assets/images/arrow.svg" /></button>
    </div>
    <div v-if="isGameEnd" class="dim">
      <div class="modal" tabindex="0" @keydown.enter="init" ref="gameEnd">
        <p>STAGE CLEAR!</p>
        <p>CLEAR TIME : {{ time }}</p>
        <button class="btn-confirm" @click="init">NEXT STAGE</button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  data () {
    return {
      mazeList: [
        [1, 1, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 0, 0, 0, 0, 0, 0, 0, 1, 0, 2],
        [1, 0, 1, 1, 1, 1, 1, 0, 1, 0, 1],
        [1, 0, 0, 0, 0, 0, 1, 0, 0, 0, 1],
        [1, 0, 1, 1, 1, 1, 1, 1, 1, 1, 1],
        [1, 0, 0, 0, 0, 0, 0, 0, 0, 0, 1],
        [1, 1, 1, 1, 1, 1, 1, 0, 1, 0, 1],
        [1, 0, 0, 0, 1, 0, 0, 0, 1, 0, 1],
        [1, 0, 1, 0, 1, 0, 1, 0, 1, 0, 1],
        [1, 0, 1, 0, 0, 0, 1, 0, 1, 0, 1],
        [1, 3, 1, 1, 1, 1, 1, 1, 1, 1, 1]
      ],
      maze: null,
      position: { x: 1, y: 10 },
      lookingLeft: false,
      score: 0,
      isGameEnd: false,
      timer: null,
      time: 0,
      isGameReady: true
    }
  },
  created () {

  },
  mounted () {

  },
  updated () {
    if (this.isGameEnd) {
      this.$refs.gameEnd.focus()
    } else {
      this.$refs.game.focus()
    }
  },
  methods: {
    init () {
      this.maze = []

      this.mazeList.forEach((array) => {
        this.maze.push([...array])
      })

      this.isGameEnd = false
      this.lookingLeft = false
      this.position = { x: 1, y: 10 }
      this.time = 0
      this.isGameReady = false

      let carrotCount = 3

      while (carrotCount > 0) {
        const randomNumber = Math.floor(Math.random() * 100)

        if (this.maze[Math.floor(randomNumber / 10)][randomNumber % 10] === 0) {
          carrotCount -= 1
          this.maze[Math.floor(randomNumber / 10)][randomNumber % 10] = 4
        }
      }

      this.timer = setInterval(() => { this.time += 1 }, 1000)
    },
    directionEvent (e) {
      const { x, y } = this.position
      let { _x, _y } = { _x: x, _y: y }

      switch (e.key) {
        case 'ArrowLeft':
          this.lookingLeft = true
          if (x - 1 >= 0 && this.maze[y][x - 1] !== 1) {
            _x -= 1
          }
          break
        case 'ArrowRight':
          this.lookingLeft = false
          if (x + 1 <= 10 && this.maze[y][x + 1] !== 1) {
            _x += 1
          }
          break
        case 'ArrowUp':
          if (y - 1 >= 0 && this.maze[y - 1][x] !== 1) {
            _y -= 1
          }
          break
        case 'ArrowDown':
          if (y + 1 <= 10 && this.maze[y + 1][x] !== 1) {
            _y += 1
          }
          break
        default:
          break
      }

      if (this.maze[_y][_x] === 4) this.score += 10
      if (this.maze[_y][_x] === 2) {
        this.isGameEnd = true
        clearInterval(this.timer)
      }

      this.maze[y][x] = 0
      this.maze[_y][_x] = 3

      this.position.x = _x
      this.position.y = _y

      const arr = [...this.maze]
      this.maze = arr
    },
    handlePressLeft () {
      this.directionEvent({key: 'ArrowLeft'})
    },
    handlePressRight () {
      this.directionEvent({key: 'ArrowRight'})
    },
    handlePressUp () {
      this.directionEvent({key: 'ArrowUp'})
    },
    handlePressDown () {
      this.directionEvent({key: 'ArrowDown'})
    }
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.maze{
  outline: none;
  font-family: 'Sunflower', sans-serif;
  font-weight: 500;
}
.maze-con > * {
  width: 30px;
  height: 30px;
  display:inline-block;
}
.left{
  transform:rotate(0deg);

  -moz-transform: scaleX(-1);

  -o-transform: scaleX(-1);

  -webkit-transform: scaleX(-1);

  transform: scaleX(-1);
}
.dim{
  background-color: rgba(255, 255, 255, .7);
  width: 100vw;
  height: 100vh;
  position: fixed;
  top: 0;
  left: 0;
}
.modal{
  display: inline-block;

  background-color: white;
  border: 1px solid #F7E3CC;
  border-radius: 10px;
  width: 30%;
  min-width: 300px;

  padding: 20px 0 30px;

  position: absolute;
  top: 50%;
  left: 50%;

  box-shadow: 0 0 15px #edebe9;

  transform: translate(-50%, -50%);

  outline: none;
}
.btn-confirm{
  font-family: 'Sunflower', sans-serif;
  font-weight: 500;
  border: none;
  margin-top: 15px;
  padding: 0px 20px 0px 20px;
  background-color: #F7E3CC;
  border-radius: 6px;
  outline: none;
  height: 35px;
  line-height: 37px;
}
.arrow-wrap{
  background:#ff7b79;
  width:150px;
  height:150px;
  border-radius: 100px;
  position: relative;
  margin:20px auto;
}
.btn-arrow{
  width:50px;
  height:50px;
  border:none;
  background:transparent;
  position:absolute;

  outline: none;
}
.btn-arrow.left{
  left:0;
  top:calc(50% - 25px);
}
.btn-arrow.right{
  right:0;
  top:calc(50% - 25px);
}
.btn-arrow.top{
  left:calc(50% - 25px);
  top:0;
  transform:rotate(-90deg);
}
.btn-arrow.bottom{
  left:calc(50% - 25px);
  bottom:0;
  transform:rotate(90deg);
}
.btn-arrow img{
  filter: invert(1);
}
</style>
