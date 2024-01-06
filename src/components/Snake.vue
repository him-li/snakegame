<template>
  <v-container class="fill-height">
    <v-responsive class="align-center text-center fill-height">
      <v-sheet>
        <div v-for="(e, i) in frame" :key="i" class="row">
          <p v-for="(b, j) in e" :key="j" class="element" :style="{ background: b.backgroundColor }">
          </p>
        </div>
      </v-sheet>
      <v-row>
        <v-col cols="12">
          <v-btn @click="moveUp()">Move Up</v-btn>
        </v-col>
        <v-col cols="4">
          <v-btn @click="moveLeft()">Move Left</v-btn>
        </v-col>
        <v-col col="4" class="control">
          <v-btn class="ztks" @click="autoMove()">
            Pause/Start
          </v-btn>
        </v-col>
        <v-col cols="4">
          <v-btn @click="moveRight()">Move Right</v-btn>
        </v-col>
        <v-col cols="12">
          <v-btn @click="moveDown()">Move Down</v-btn>
        </v-col>
      </v-row>
      <v-row>
        <v-col cols="6">
          <p>Level</p>
          <p>{{ level }}</p>
        </v-col>
        <v-col cols="6">
          <p>Score</p>
          <p>{{ times }}</p>
        </v-col>
      </v-row>
    </v-responsive>
  </v-container>
</template>

<script>
export default {
  data() {
    return {
      row: 30,
      col: 30,
      frame: [],
      backgroundColor: "#eee",
      snake: {},
      food: {},
      color: 'teal',
      over: false,
      timer: '',
      speed: 600,
      fx: 0,
      times: 0,
      level: 1,
      site: 0,//食物位置
    }
  },
  methods: {
    gameFrame() {
      for (let i = 0; i < this.row; i++) {
        let a = [];
        for (let j = 0; j < this.col; j++) {
          let b = {
            backgroundColor: this.backgroundColor,
          };
          a.push(b);
        }
        this.frame.push(a);
      }
    },

    initSnake() {
      this.snake = {
        site: [9, 8, 9, 9, 9, 10],
        color: this.color,
      }
      this.renderBlock(this.snake, this.frame, 1);
    },

    //方块渲染
    //a:方块，b:渲染的位置,n类型：0清除，1渲染
    renderBlock(a, b, n) {
      let c = a.site;
      if (n == 1) {
        for (let i = 0; i < c.length; i += 2) {
          b[c[i]][c[i + 1]].backgroundColor = a.color;
        }
      } else if (n == 0) {
        for (let i = 0; i < c.length; i += 2) {
          b[c[i]][c[i + 1]].backgroundColor = this.backgroundColor;
        }
      }
    },

    initFood() {
      this.sfood();
      let site = this.site;
      let color = this.color[Math.floor(Math.random() * 7)]
      this.food = { site, color };
      this.renderBlock(this.food, this.frame, 1);
    },

    sfood() {
      this.site = [Math.floor(Math.random() * this.row), Math.floor(Math.random() * this.col)];
      for (let i = 0; i < this.snake.site.length; i += 2) {
        if (this.snake.site[i] == this.site[0] && this.snake.site[i + 1] == this.site[1]) {
          this.sfood();
        }
      }
    },

    move() {
      for (let i = this.snake.site.length - 1; i > 1; i -= 2) {
        this.snake.site[i] = this.snake.site[i - 2];
        this.snake.site[i - 1] = this.snake.site[i - 3];
      }
    },

    moveLeft() {
      if (!this.over && this.fx != 1) {
        this.fx = 0;
        clearInterval(this.timer);
        this.timer = setInterval(() => {
          this.renderBlock(this.snake, this.frame, 0);
          this.eat(this.snake.site[0], this.snake.site[1] - 1);
          this.move();
          this.snake.site[1]--;
          if (this.snake.site[1] < 0) {
            this.over = true;
            console.log('撞墙了')
            this.snake.site[1]++;
            this.oneself();
            clearInterval(this.timer);
          }
          this.renderBlock(this.snake, this.frame, 1);
        }, this.speed)

      }
    },

    moveRight() {
      if (!this.over && this.fx != 0) {
        this.fx = 1;
        clearInterval(this.timer);
        this.timer = setInterval(() => {
          this.renderBlock(this.snake, this.frame, 0);
          this.eat(this.snake.site[0], this.snake.site[1] + 1);
          this.move();
          this.snake.site[1]++;
          if (this.snake.site[1] >= this.col) {
            this.over = true;
            console.log('撞墙了')
            this.snake.site[1]--;
            this.oneself();
            clearInterval(this.timer);
          }
          this.renderBlock(this.snake, this.frame, 1);
        }, this.speed)
      }
    },

    moveUp() {
      if (!this.over && this.fx != 3) {
        this.fx = 2;
        clearInterval(this.timer);
        this.timer = setInterval(() => {
          this.renderBlock(this.snake, this.frame, 0);
          this.eat(this.snake.site[0] - 1, this.snake.site[1]);
          this.move();
          this.snake.site[0]--
          if (this.snake.site[0] < 0) {
            this.over = true;
            console.log('撞墙了')
            this.snake.site[0]++;
            this.oneself();
            clearInterval(this.timer);
          }
          this.renderBlock(this.snake, this.frame, 1);
        }, this.speed)
      }
    },

    moveDown() {
      if (!this.over && this.fx != 2) {
        this.fx = 3;
        clearInterval(this.timer);
        this.timer = setInterval(() => {
          this.renderBlock(this.snake, this.frame, 0);
          this.eat(this.snake.site[0] + 1, this.snake.site[1]);
          this.move();
          this.snake.site[0]++
          this.oneself();
          if (this.snake.site[0] >= this.row) {
            this.over = true;
            console.log('撞墙了')
            this.snake.site[0]--;
            clearInterval(this.timer);
          }
          this.renderBlock(this.snake, this.frame, 1);
        }, this.speed)
      }
    },
    //碰到自己
    oneself() {
      //拿当前头部来与身体对比
      let t = [this.snake.site[0], this.snake.site[1]];
      for (let i = this.snake.site.length - 1; i > 1; i -= 2) {
        if (this.snake.site[i] == t[1] && this.snake.site[i - 1] == t[0]) {
          clearInterval(this.timer);
          this.over = true;
          console.log('碰到自己了')
        }
      }
    },
    //吃
    eat(i, j) {
      if (i == this.food.site[0] && j == this.food.site[1]) {
        this.snake.site.unshift(this.food.site[0], this.food.site[1]);
        this.initFood();
        this.times++;

        let lev = Math.floor(this.times / 10) + 1;

        if (lev > this.level) {
          this.level = lev;
          if (this.level < 15) {
            this.speed = 600 - (this.level - 1) * 40;
          } else {
            this.speed = 30;
          }
          clearInterval(this.timer);
          this.autoMove();
        }
      }

    },

    autoMove() {
      if (this.timer) {
        clearInterval(this.timer);
        this.timer = '';
      } else {
        if (this.fx == 0) {
          this.moveLeft();
        } else if (this.fx == 2) {
          this.moveUp();
        } else if (this.fx == 1) {
          this.moveRight();
        } else {
          this.moveDown();
        }
      }
    }
  },
  mounted() {
    this.gameFrame();
    this.initSnake();
    this.initFood();
    this.autoMove();
  }
}
</script>

<style scoped>
p {
  padding: 0;
  margin: 0;
}

.row {
  display: flex;
  padding-bottom: 1px;
}

.element {
  height: 12px;
  width: 12px;
  margin-right: 1px;
}
</style>