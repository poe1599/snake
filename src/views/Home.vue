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
    this.start();
    setInterval(() => {
      this.snakeMove();
    }, 20);
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
    start() {
      this.snake.pos = [];
      this.snake.pos.push(this.ranPos());
      this.setBug();
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
        this.start();
        return;
      }

      let newSnakeHead = null;
      let d = Math.abs(Sx - Bx) + Math.abs(Sy - By);
      headList2.forEach((i) => {
        const newD = Math.abs(i.x - Bx) + Math.abs(i.y - By);
        if (newD < d) newSnakeHead = i;
      });
      if (newSnakeHead === null) newSnakeHead = headList2[0];

      this.snake.pos.unshift(newSnakeHead);
      this.snake.pos.pop();
    },
    state(x, y) {
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
}

.s {
  // &__wrap {
  // }

  &__row {
    display: flex;
  }

  &__li {
    &--snake {
      background: rgb(253, 253, 253);
    }

    &--snakeHead {
      background: rgb(255, 222, 212);
    }

    &--bug {
      background: rgb(14, 233, 7);
    }
  }
}
</style>
