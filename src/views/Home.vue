<template>
  <div class="container">
    <div class="s">
      <div class="s__wrap">
        <div class="s__row" v-for="y in Y" :key="`Y:(${y})`">
          <div
            :data="`Y:(${y})X:(${x})`"
            class="s__li"
            :class="state(x, y)"
            v-for="x in X"
            :key="`Y:(${y})X:(${x})`"
            :style="{ height: D, width: D }"
          ></div>
        </div>
      </div>
    </div>

    <div class="control">
      <button
        class="control__toggle"
        :class="{ 'control__toggle--active': isControlClose }"
        @click="isControlClose = !isControlClose"
      ></button>
      <div
        class="control__wrap"
        :class="{ 'control__wrap--close': isControlClose }"
      >
        <button
          class="control__btn"
          :class="`${isLoop ? 'control__btn--isLoop' : 'control__btn--once'}`"
          @click="isLoop = !isLoop"
        ></button>
        <button
          class="control__btn control__btn--reset"
          @click="reset"
        ></button>
      </div>
    </div>
  </div>
</template>

<script>
export default {
  name: "Home",
  components: {},
  data() {
    return {
      WH: 0,
      WW: 0,
      X: 40,
      Y: 40,
      snake: {
        pos: [],
      },
      bug: { x: 0, y: 0 },
      intervalCode: null,
      isLoop: true,
      isOnReset: false,
      isControlClose: true,
    };
  },
  computed: {
    D() {
      const h = Math.floor(this.WH / this.Y);
      const w = Math.floor(this.WW / this.X);
      if (h > w) return `${w}px`;
      return `${h}px`;
    },
  },
  mounted() {
    window.addEventListener("resize", this.resize);
    this.resize();
    this.init();
    this.start();
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.resize);
  },
  methods: {
    resize() {
      this.WH = window.innerHeight;
      this.WW = window.innerWidth;
    },
    // 產生隨機座標
    ranPos() {
      return {
        x: Math.floor(Math.random() * this.X) + 1,
        y: Math.floor(Math.random() * this.Y) + 1,
      };
    },
    init() {
      this.snake.pos = [];
      this.snake.pos.push(this.ranPos());
      this.setBug();
    },
    start() {
      this.isOnReset = false;
      this.intervalCode = setInterval(() => {
        this.snakeMove();
      }, 41.6);
    },
    stop() {
      clearInterval(this.intervalCode);
    },
    reset() {
      this.stop();
      this.init();
      this.start();
    },
    setBug() {
      this.bug = this.ranPos();
      const { x, y } = this.bug;
      const isBugInSnake = this.snake.pos.some((i) => i.x === x && i.y === y);
      if (isBugInSnake) this.setBug();
    },
    snakeMove() {
      const Sx = this.snake.pos[0].x;
      const Sy = this.snake.pos[0].y;
      const Bx = this.bug.x;
      const By = this.bug.y;

      // 吃到bug
      if (Sx === Bx && Sy === By) {
        this.setBug();
        this.snake.pos.push({ x: Sx, y: By });
        return;
      }

      // 下個蛇頭位
      const headList1 = [
        { x: Sx + 1, y: Sy },
        { x: Sx - 1, y: Sy },
        { x: Sx, y: Sy + 1 },
        { x: Sx, y: Sy - 1 },
      ];

      // 下個合法蛇頭位
      const headList2 = [];
      headList1.forEach((i) => {
        const { x, y } = i;
        const isInSpace = x >= 1 && x <= this.X && y >= 1 && y <= this.Y;
        if (!isInSpace) return;
        const isInSnake = this.snake.pos.some((j) => j.x === x && j.y === y);
        if (isInSnake) return;
        headList2.push(i);
      });

      // 不存在合法蛇頭則重來
      if (headList2.length === 0) {
        this.stop();
        if (!this.isLoop) return;
        this.isOnReset = true;
        setTimeout(() => {
          this.reset();
        }, 5000);
        return;
      }

      let newSnakeHead = null;
      let d = Math.abs(Sx - Bx) + Math.abs(Sy - By);
      headList2.forEach((i) => {
        const newD = Math.abs(i.x - Bx) + Math.abs(i.y - By);
        if (newD < d) newSnakeHead = i;
      });
      if (newSnakeHead === null)
        newSnakeHead = headList2[Math.floor(Math.random() * headList2.length)];

      this.snake.pos.unshift(newSnakeHead);
      this.snake.pos.pop();
    },
    state(x, y) {
      if (this.isOnReset) return "s__li--onReset";
      if (this.snake.pos.length === 0) return "";
      const isSnake = this.snake.pos.some((i) => i.x === x && i.y === y);
      const isSnakeHead =
        this.snake.pos[0].x === x && this.snake.pos[0].y === y;
      const isBug = this.bug.x === x && this.bug.y === y;
      if (isSnakeHead) return "s__li--snakeHead";
      if (isSnake) return "s__li--snake";
      if (isBug) return "s__li--bug";
      return "";
    },
  },
};
</script>
<style lang="scss" scoped>
.container {
  width: 100%;
  height: 100vh;
  background: rgb(124, 120, 120);
  display: flex;
  justify-content: center;
  align-items: center;
  background: url("~@/assets/space.jpg") center center no-repeat;
  background-size: cover;
  position: relative;
}

.s {
  // &__wrap {
  // }

  &__row {
    display: flex;
  }

  &__li {
    transition: 0.5s;
    &--snake {
      background: rgb(252, 223, 214);
    }

    &--snakeHead {
      background: rgb(253, 74, 19);
      transition: 0s;
    }

    &--bug {
      background: rgb(9, 255, 0);
    }

    &--onReset {
      transition: 5s;
    }
  }
}

.control {
  position: absolute;
  top: 5vh;
  right: 5vw;
  width: 50px;
  text-align: center;

  &__toggle {
    display: inline-block;
    width: 20px;
    height: 20px;
    border-radius: 50%;
    padding: 0;
    transition: 1s;
    transform: translateY(70vh);
    margin-bottom: 16px;
    cursor: pointer;

    &--active {
      transform: translateY(0vh);
    }
  }

  &__wrap {
    overflow: hidden;
    transition: 0.5s;
    max-height: 100vh;

    &--close {
      max-height: 0;
    }
  }

  &__btn {
    width: 100%;
    height: 2vw;
    margin-bottom: 2vw;
    text-align: center;
    padding: 0;
    transition: 0.3s;
    background: #000;
    color: chocolate;
    min-height: 30px;
    cursor: pointer;

    &--isLoop {
      &::before {
        content: "Loop";
      }
    }

    &--once {
      &::before {
        content: "Once";
      }
    }

    &--reset {
      &::before {
        content: "Reset";
      }
    }
  }

  &__btn:hover {
    @extend %BtnHover;
  }
}
%BtnHover {
  background: rgb(253, 74, 19);
  color: black;
}
</style>
