<template>
  <div v-if="isGameReady">
    <img class="logo" src="../assets/images/home.svg">
    <p class="title">길토배고</p>
    <button class="btn-confirm" @click="isRandom = false; init()">NORMAL MODE</button>
    <br/>
    <button class="btn-confirm" @click="isRandom = true; init()">RANDOM MODE</button>
  </div>
  <div v-else class="maze" @keydown="directionEvent" tabindex="0" ref="game">
    <div v-for='(row, index) in maze' v-bind:key='index' class="maze-con">
      <template v-for='(col, _index) in row'>
        <span v-if="col === 0" v-bind:key="'_' + _index"></span>
        <img v-else-if="col === 1" src="../assets/images/bush.svg" v-bind:key="'_' + _index">
        <img v-else-if="col === 2" src="../assets/images/home.svg" v-bind:key="'_' + _index">
        <span ref="rabbit" class="rabbit-wrap" v-else-if="col === 3" v-bind:key="'_' + _index"><img :class="lookingLeft ? 'left' : null" src="../assets/images/rabbit.gif"></span>
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
      maze: null,
      position: { x: 1, y: 1 },
      lookingLeft: false,
      score: 0,
      isGameEnd: false,
      timer: null,
      time: 0,
      isGameReady: true,
      isRandom: false,
      size: 11,
      carrotCount: 0,
      eatCarrotCount: 0
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
      if (this.isRandom) {
        switch (Math.floor(Math.random() * 3)) {
          case 0:
            this.size = 9
            break
          case 1:
            this.size = 11
            break
          case 2:
            this.size = 13
            break
        }
      }

      this.maze = []

      this.display(this._maze(this.size, this.size)).forEach((row) => {
        this.maze[row[0].y] = Array(this.size)
        row.forEach((col) => {
          this.maze[col.y][col.x] = col.type === 'block' ? 1 : (col.type === 'finish' ? 2 : 0)
        })
      })

      this.maze[1][1] = 3

      // this.mazeList.forEach((array) => {
      //   this.maze.push([...array])
      // })

      this.isGameEnd = false
      this.lookingLeft = false
      this.position = { x: 1, y: 1 }
      this.time = 0
      this.isGameReady = false

      switch (Math.floor(Math.random() * 3)) {
        case 0:
          this.carrotCount = 2
          break
        case 1:
          this.carrotCount = 4
          break
        case 2:
          this.carrotCount = 6
          break
      }

      this.eatCarrotCount = this.carrotCount

      while (this.eatCarrotCount > 0) {
        const randomNumber = Math.floor((Math.random() * (((this.size - 1) * (this.size - 1)) - (this.size + 1) + 1)) + this.size + 1)

        if (this.maze[Math.floor(randomNumber / (this.size))][randomNumber % (this.size - 1)] === 0) {
          this.eatCarrotCount -= 1
          this.maze[Math.floor(randomNumber / (this.size))][randomNumber % (this.size - 1)] = 4
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
          if (x + 1 < this.size && this.maze[y][x + 1] !== 1) {
            if (this.maze[y][x + 1] === 2) {
              if (this.eatCarrotCount >= this.carrotCount) {
                _x += 1
              }
            } else {
              _x += 1
            }
          }
          break
        case 'ArrowUp':
          if (y - 1 >= 0 && this.maze[y - 1][x] !== 1) {
            _y -= 1
          }
          break
        case 'ArrowDown':
          if (y + 1 < this.size && this.maze[y + 1][x] !== 1) {
            _y += 1
          }
          break
        default:
          break
      }

      if (this.maze[_y][_x] === 4) {
        this.score += 10
        this.eatCarrotCount += 1
      }
      if (this.maze[_y][_x + 1] === 2 && e.key === 'ArrowRight') {
        if (this.eatCarrotCount < this.carrotCount) {
          this.$refs.rabbit[0].classList.add('active')
        }
      }
      if (this.maze[_y][_x] === 2) {
        if (this.eatCarrotCount === this.carrotCount) {
          this.isGameEnd = true
          clearInterval(this.timer)
        }
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
    },
    _maze (inputX, inputY) {
      const x = (inputX - 1) / 2
      const y = (inputY - 1) / 2

      let j
      let n = x * y - 1

      if (n < 0) {
        // eslint-disable-next-line no-alert
        alert('illegal maze dimensions')

        return
      }

      const horiz = []

      for (j = 0; j < x + 1; j += 1) {
        horiz[j] = []
      }

      const verti = []

      for (j = 0; j < x + 1; j += 1) {
        verti[j] = []
      }

      let here = [Math.floor(Math.random() * x), Math.floor(Math.random() * y)]
      const path = [here]
      const unvisited = []

      for (j = 0; j < x + 2; j += 1) {
        unvisited[j] = []

        for (let k = 0; k < y + 1; k += 1) {
          unvisited[j]
            .push(j > 0 && j < x + 1 && k > 0 && (j !== here[0] + 1 || k !== here[1] + 1))
        }
      }

      while (n > 0) {
        const potential = [
          [here[0] + 1, here[1]], [here[0], here[1] + 1],
          [here[0] - 1, here[1]], [here[0], here[1] - 1]
        ]

        const neighbors = []

        for (j = 0; j < 4; j += 1) {
          if (unvisited[potential[j][0] + 1][potential[j][1] + 1]) {
            neighbors.push(potential[j])
          }
        }

        if (neighbors.length) {
          n -= 1

          const next = neighbors[Math.floor(Math.random() * neighbors.length)]

          unvisited[next[0] + 1][next[1] + 1] = false

          if (next[0] === here[0]) {
            horiz[next[0]][(next[1] + here[1] - 1) / 2] = true
          } else {
            verti[(next[0] + here[0] - 1) / 2][next[1]] = true
          }

          path.push(here = next)
        } else {
          here = path.pop()
        }
      }

      // eslint-disable-next-line consistent-return
      return {
        x, y, horiz, verti
      }
    },
    display (m) {
      const mazeArea = []

      for (let y = 0; y < m.x * 2 + 1; y += 1) {
        const row = []

        if (y % 2 === 0) {
          for (let x = 0; x < m.y * 2 + 1; x += 1) {
            if (x % 2 === 0) {
              row.push({ type: 'block', x, y })
            } else if (y > 0 && m.verti[y / 2 - 1][Math.floor(x / 2)]) {
              row.push({ type: null, x, y })
            } else {
              row.push({ type: 'block', x, y })
            }
          }
        } else {
          for (let x = 0; x < m.y * 2 + 1; x += 1) {
            if (x % 2 === 0) {
              if (x > 0 && m.horiz[(y - 1) / 2][x / 2 - 1]) {
                row.push({ type: null, x, y })
              } else {
                row.push({ type: 'block', x, y })
              }
            } else {
              row.push({ type: null, x, y })
            }
          }
        }

        // start
        // if (y === 0) {
        //   row[1] = '1';
        // }

        // end
        if (m.x * 2 - 1 === y) {
          row[2 * m.y] = { type: 'finish', x: 2 * m.y, y }
        }

        mazeArea.push(row)
      }

      return mazeArea.slice().reverse()
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

.rabbit-wrap{
  position:relative;
}
.rabbit-wrap img{
  width:24px;
  margin:3px;
}
.rabbit-wrap.active:after{
  content:'아직 배고파요!';
  display: block;
  padding: 5px 10px;
  background: rgba(255,255,255,.7);
  border: 1px solid #ccc;
  border-radius: 8px;
  position: absolute;
  left: -43px;
  top: -35px;
  width: -webkit-fit-content;
  width: -moz-fit-content;
  width: max-content;
}
</style>
