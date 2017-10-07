粘贴一个贪食蛇的代码

    <template>
      <div :style="{width: canvasWidth + 'px'}" class="game-panel">
        <canvas :width="canvasWidth" :height="canvasHeight" ref="canvas"></canvas>
        <span class="score">{{score}}</span>
        <button class="btn-start" @click="start" v-if="!isStart && !isEnd">开始游戏</button>
        <div class="result" v-if="isEnd">
          <p class="title">GAME OVER</p>
          <p>本轮得分：{{score}}</p>
          <button class="btn-restart" @click="start">再来一局</button>
        </div>
      </div>
    </template>

    <script>
      
      const BLANK = 0;
      const FRUIT = 1;
      const SNAKE_HEAD = 2;
      const SNAKE_BODY = 3;
      const WALL = 4;

      const UP = 0;
      const DOWN = 1;
      const LEFT = 2;
      const RIGHT = 3;

      const INTERVAL = 250;

      const debug = false;

      export default {
        props: {
          width: {
            type: Number,
            default: 64,
          },
          height: {
            type: Number,
            default: 48,
          }
        },
        data () {
          return {
            isStart: false,
            isEnd: false,
            score: 0,
            fruits: 0,

            map: [],
            snake: [],

            mapWidth: this.width,
            mapHeight: this.height,

            direction: UP,
            timer: -1,
          }
        },
        computed: {
          canvasWidth () {
            return this.width * 10;
          },
          canvasHeight () {
            return this.height * 10;
          },
          isPlaying () {
            return this.isStart && !this.isEnd;
          }
        },
        methods: {

          state (x, y, state = null) {
            if(state === null) {
              return this.map[y][x];
            } else {
              this.map[y][x] = state;

              let canvas = this.$refs.canvas;
              let ctx = canvas.getContext('2d');
              switch (state) {
                case BLANK:
                  ctx.fillStyle = 'white';
                  ctx.fillRect(x * 10, y * 10, 10, 10);
                  break;
                case FRUIT:
                  ctx.fillStyle = 'orange';
                  ctx.beginPath();
                  ctx.arc(x * 10 + 5, y * 10 + 5, 5, 0, Math.PI * 2, true);
                  ctx.closePath();
                  ctx.fill();
                  break;
                case SNAKE_HEAD:
                  ctx.fillStyle = 'green';
                  ctx.fillRect(x * 10, y * 10, 10, 10);
                  break;
                case SNAKE_BODY:
                  ctx.fillStyle = 'lightgreen';
                  ctx.fillRect(x * 10, y * 10, 10, 10);
                  break;
                case WALL:
                  ctx.fillStyle = 'gray';
                  ctx.fillRect(x * 10, y * 10, 10, 10);
                  break;
              }

              return state;
            }
          },

          clearCanvas () {
            let canvas = this.$refs.canvas;
            let ctx = canvas.getContext('2d');
            ctx.fillStyle = 'white';
            ctx.fillRect(0, 0, this.canvasWidth, this.canvasHeight);
          },

          start () {
            this.isStart = true;
            this.isEnd = false;
            this.fruits = 0;
            this.score = 0;
            this.direction = UP;

            // 创建一个空画布
            this.map = new Array(this.mapHeight);
            for(let i = 0; i < this.mapHeight; i++){
              this.map[i] = new Array(this.mapWidth);
              for(let j = 0; j < this.mapWidth; j++){
                this.map[i][j] = BLANK;
              }
            }
            this.clearCanvas();

            // 创建蛇
            this.snake = new Array();
            let startX = Math.floor(this.mapWidth / 2);
            let startY = Math.floor(this.mapHeight / 2) - 1;
            this.snake.push({x: startX, y: startY});
            this.snake.push({x: startX, y: startY + 1});
            this.snake.push({x: startX, y: startY + 2});
            for(let v of this.snake){
              this.state(v.x, v.y, SNAKE_BODY);
            }
            let snakeHead = this.snake[0];
            this.state(snakeHead.x, snakeHead.y, SNAKE_HEAD);

            // 创建第一个果子
            this.createFruit();

            // 开始游戏状态！
            this.timer = setInterval(this.nextStep, INTERVAL);
          },

          nextStep () {
            if(!this.isPlaying) return false;
            let snakeHead = this.snake[0];
            let nextPoint = {x: snakeHead.x, y: snakeHead.y};
            switch (this.direction) {
              case UP:
                nextPoint.y--;
              break;
              case DOWN:
                nextPoint.y++;
              break;
              case LEFT:
                nextPoint.x--;
              break;
              case RIGHT:
                nextPoint.x++;
              break;
            }

            // 撞到四周的墙，游戏结束
            if(nextPoint.x < 0 || nextPoint.x >= this.mapWidth
              || nextPoint.y < 0 || nextPoint.y >= this.mapHeight){
              this.gameOver();
              return false;
            }

            let state = this.state(nextPoint.x, nextPoint.y);

            // 撞到墙壁或身体，游戏结束
            if(state !== FRUIT && state !== BLANK){
              this.gameOver();
              return false;
            }

            // 向前增长一格
            this.state(nextPoint.x, nextPoint.y, SNAKE_HEAD);
            for(let v of this.snake){
              this.state(v.x, v.y, SNAKE_BODY);
            }
            this.snake.unshift({x: nextPoint.x, y: nextPoint.y});

            // 如果不是吃到水果，尾巴缩一格，实际相当于前进
            if(state !== FRUIT){
              let snakeTail = this.snake.pop();
              this.state(snakeTail.x, snakeTail.y, BLANK);
            }else{
              // 吃到水果，生成下一个水果
              this.fruits++;
              this.score += this.fruits;
              this.createFruit();
            }

            return true;
          },

          createFruit () {
            let x = Math.floor(Math.random() * this.mapWidth);
            let y = Math.floor(Math.random() * this.mapHeight);
            if(this.state(x, y) == BLANK){
              this.state(x, y, FRUIT);
              return {x: x, y: y};
            } else {
              return this.createFruit();
            }
          },

          gameOver () {
            clearInterval(this.timer);
            this.timer = -1;
            this.isEnd = true;
            this.isStart = false;
          },

        },
        created: function(){
          document.addEventListener('keydown', e => {
            e = e || event;
            if(!this.isPlaying) return;
            switch(e.keyCode){
              case 37: if(this.direction !== RIGHT) this.direction = LEFT; else return; break;
              case 38: if(this.direction !== DOWN) this.direction = UP; else return; break;
              case 39: if(this.direction !== LEFT) this.direction = RIGHT; else return; break;
              case 40: if(this.direction !== UP) this.direction = DOWN; else return; break;
            }
            // 将下一步提前进行
            if(e.keyCode >= 37 && e.keyCode <= 40){
              clearInterval(this.timer);
              if(this.nextStep()){
                this.timer = setInterval(this.nextStep, INTERVAL);
              }
            }
          });
        }
      }


    </script>

    <style lang="less" scoped>
      canvas {
        position: relative;
        z-index: 1;
        box-sizing: content-box;
        border: 10px solid gray;
        background-color: white;
        box-shadow: 0 0 17px 1px #A9AAA6;
      }

      div.game-panel {
        position: relative;
        margin: 0 auto;

        .score {
          position: absolute;
          top: 12px;
          left: 18px;
          z-index: 2;
          font-size: 16px;
          font-family: Consolas;
          letter-spacing: 2px;
          font-weight: bold;
          color: #840000;
          opacity: 0.6;
          pointer-events: none;
        }

        button {
          width: 200px;
          height: 50px;
          font-size: 24px;
          color: white;
          text-align: center;
          line-height: 50px;
          background-color: #F86D3F;
          border-radius: 5px;
          transition: all .2s;
          cursor: pointer;
          border: none;

          &:hover {
            background-color: #F4541E;
          }

          &.btn-start {
            position: absolute;
            top: 120px;
            left: ~"calc((100% + 20px - 200px) / 2)";
            z-index: 3;
          }
        }

        div.result {
          position: absolute;
          top: 40px;
          width: 340px;
          left: ~"calc((100% + 20px - 340px) / 2)";
          z-index: 3;
          text-align: center;
          padding: 40px 20px;
          background-color: fade(#FFF, 50%);

          p {
            margin-bottom: 15px;
            font-size: 16px;

            &.title {
              font-weight: bold;
              color: #910048;
              font-size: 36px;
            }
          }
        }
      }

    </style>
