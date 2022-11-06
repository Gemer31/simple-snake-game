<template>
  <div class="game">
    <canvas
        class="canvas"
        id="canvas"
        width="640"
        height="640"
    ></canvas>
    <div v-if="!gameInterval" class="game-over">
      <span>{{ message }}</span>
      <button class="game-over-button" type="button" @click.prevent="startGame">Try again</button>
    </div>

    <img :hidden="true" id="groundImg" src="./assets/ground.png">
    <img :hidden="true" id="foodImg" src="./assets/apple.svg">
    <img :hidden="true" id="snakeBody" src="./assets/snake-body.svg">
    <img :hidden="true" id="snakeHead" src="./assets/snake-head.svg">
  </div>
</template>

<script>
import defaultSnake from "./constants/snake-default"

export default {
  name: 'App',
  data() {
    return {
      message: "Game Over",
      cellSize: 32,
      rowCellCount: 20,
      gameInterval: null,
      snake: [],
      addSnakeEl: false,
      foodPosition: null,
      direction: "up",
      context: null,
    }
  },
  methods: {
    getHeadNewPosition(el) {
        let newXY;
        switch (this.direction) {
          case "up": {
            newXY = el.y - 1;
            return {
              x: el.x,
              y: newXY < 0 ? 19 : newXY,
            };
          }
          case "down": {
            newXY = el.y + 1;
            return {
              x: el.x,
              y: newXY > 19 ? 0 : newXY,
            };
          }
          case "left": {
            newXY = el.x - 1;
            return {
              x: newXY < 0 ? 19 : newXY,
              y: el.y,
            };
          }
          case "right": {
            newXY = el.x + 1;
            return {
              x: newXY > 19 ? 0 : newXY,
              y: el.y,
            };
          }
          default: break;
        }
    },
    getFoodNewPosition() {
      const newFoodPosition = {
        x: Math.floor(Math.random() * this.rowCellCount),
        y: Math.floor(Math.random() * this.rowCellCount),
      };
      const isDuplicatePosition = this.snake.some((snakeEl) => {
        return snakeEl.x === newFoodPosition.x && snakeEl.y === newFoodPosition.y;
      })
      return isDuplicatePosition
          ? this.getFoodNewPosition()
          : newFoodPosition;
    },
    isGameOver() {
      return this.snake.slice(1).some((bodyEl) => {
        const head = this.snake[0];
        return bodyEl.x === head.x && bodyEl.y === head.y;
      });
    },
    isGameWin() {
      return this.snake.length === (this.rowCellCount * this.rowCellCount - 1);
    },
    drawGame() {
      if (!this.isGameWin()) {
        this.context.drawImage(document.getElementById("groundImg"), 0, 0, 640, 640);

        this.snake = [
          this.getHeadNewPosition(this.snake[0]),
          ...(this.addSnakeEl ? this.snake : this.snake.slice(0, this.snake.length - 1)),
        ];
        this.addSnakeEl = false;

        if (this.isGameOver()) {
          clearInterval(this.gameInterval);
          this.gameInterval = null;
        } else {
          if (this.snake[0].x === this.foodPosition.x && this.snake[0].y === this.foodPosition.y) {
            this.addSnakeEl = true;
            this.foodPosition = this.getFoodNewPosition();
          }

          this.context.drawImage(
              document.getElementById("foodImg"),
              this.foodPosition.x * this.cellSize,
              this.foodPosition.y * this.cellSize,
              30,
              30,
          );

          this.snake.forEach((el, index) => {
            this.context.drawImage(
                document.getElementById(index === 0 ? "snakeHead" : "snakeBody"),
                el.x * this.cellSize,
                el.y * this.cellSize,
                30,
                30,
            );
          });
        }
      } else {
        clearInterval(this.gameInterval);
        this.gameInterval = null;
        this.message = "You are the best!!!";
      }
    },
    startGame() {
      this.message = "Game Over";
      this.direction = "up";
      this.snake = defaultSnake;
      this.foodPosition = this.getFoodNewPosition();
      this.gameInterval = setInterval(this.drawGame, 500);
    }
  },
  mounted() {
    window.addEventListener("keyup", (event) => {
      if (event.key === "ArrowUp") {
        if (this.direction !== "down") {
          this.direction = "up";
        }
      }
      if (event.key === "ArrowDown") {
        if (this.direction !== "up") {
          this.direction = "down";
        }
      }
      if (event.key === "ArrowLeft") {
        if (this.direction !== "right") {
          this.direction = "left";
        }
      }
      if (event.key === "ArrowRight") {
        if (this.direction !== "left") {
          this.direction = "right";
        }
      }
    });

    const canvas = document.getElementById("canvas");
    this.context = canvas.getContext("2d");

    this.startGame();
  }
}
</script>

<style>
#app {
  width: 100%;
  height: 100%;
}

.canvas {
  border: 2px solid black;
}
.game {
  height: 100%;
  display: flex;
  align-items: center;
  justify-content: center;
}

.game-over {
  background: gray;
  border-radius: 10px;
  padding: 10px;
  font-family: "Akzidenz Grotesk BQ Medium", -apple-system, BlinkMacSystemFont, sans-serif;
  font-style: oblique;
  font-size: 20px;
  display: flex;
  flex-direction: column;
  justify-content: center;
  position: absolute;
}

.game-over-button {
  margin-top: 5px;
  background-color: #13aa52;
  border: 1px solid #13aa52;
  border-radius: 4px;
  box-shadow: rgba(0, 0, 0, .1) 0 2px 4px 0;
  box-sizing: border-box;
  color: #fff;
  cursor: pointer;
  font-family: "Akzidenz Grotesk BQ Medium", -apple-system, BlinkMacSystemFont, sans-serif;
  font-size: 16px;
  font-weight: 400;
  outline: none;
  outline: 0;
  padding: 5px 20px;
  text-align: center;
  transform: translateY(0);
  transition: transform 150ms, box-shadow 150ms;
  user-select: none;
  -webkit-user-select: none;
  touch-action: manipulation;
}

.game-over-button:hover {
  box-shadow: rgba(0, 0, 0, .15) 0 3px 9px 0;
  transform: translateY(-2px);
}
</style>
