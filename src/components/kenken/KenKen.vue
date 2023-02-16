<template>
  <div>&nbsp; </div>
  <h3>Ken Ken</h3>
  <div>&nbsp; </div>
  <section class="boardDisplay">
    <canvas id="mycanvas" ref="mycanvas" width="360" height="360"></canvas>
  </section>
  <build-puzzle :current-error="currentError" @buildmode-events="buildModeEvents" v-if="buildMode"></build-puzzle>
  <solve-puzzle :current-cursor="currentCursor" @solvemode-events="solveModeEvents" v-else></solve-puzzle>

</template>

<script>
import SolvePuzzle from './SolvePuzzle.vue';
import BuildPuzzle from './BuildPuzzle.vue';

export default {
  name: 'App',
  components: {
    BuildPuzzle,
    SolvePuzzle
  },
  data(){
    return {
      cellSize: 60,
      lineWidth: 1,
      buildMode:true,
      gridCanv: null,
      currentValue: -1,
      currentCursor: [0,0],
      prevCursor: [0,0],
      currentCells: new Map(),
      gamegrid: Array(6).fill().map(() => new Array(0,0,0,0,0,0)),
      currentError: ""
    }
  },
  mounted() {
      this.gridCanv = this.$refs.mycanvas
      this.drawGrid()
      this.gridCanv.addEventListener('mousedown', (e) => {
      const cellSize = 60;
      const col = Math.floor(e.offsetX / cellSize);
      const row = Math.floor(e.offsetY / cellSize);

      const pos_x = col * cellSize + 22;
      const pos_y = row * cellSize + 40;
      console.log("click (x, y)", e.offsetX, e.offsetY);
      console.log("click (row, col)", row, col);
      this.updateGrid(pos_x, pos_y, col+1, false) 
      this.gameGridOnClick(row, col)		  
   })
  },
  methods:{
    toggleMode(){
      this.buildMode = !this.buildMode
      if (this.buildMode){
        this.reDrawGrid()
        this.drawRect(this.currentCursor[0], this.currentCursor[1], "white", 2)
      }else{
        const mapIterator = this.currentCells.values();
        let i = 0;
        while (i < this.currentCells.size) {
          let val = mapIterator.next().value
          this.drawRect(val.row, val.col, "white", 2)
          i++;
        }
        this.currentCells = new Map()
        this.drawRect(this.currentCursor[0], this.currentCursor[1], "#880017", 1)
      }
      this.currentCursor = [0, 0]
    },
    drawGrid(){
      const ctxt = this.gridCanv.getContext("2d") 
      //const lineWidth = 1;
      //main
      ctxt.strokeStyle = "#00a876";
      //draw vertical lines
      ctxt.moveTo(2, 0);
      ctxt.lineTo(2, 358);

      ctxt.moveTo(358, 2);
      ctxt.lineTo(358, 358);

      //draw horizontal lines
      ctxt.moveTo(2, 2);
      ctxt.lineTo(358, 2);

      ctxt.moveTo(2, 358);
      ctxt.lineTo(358, 358);

      ctxt.lineWidth = this.lineWidth*4;
      ctxt.stroke();

      for (var j=60; j<360; j+=60) {
        //draw vertical lines
        ctxt.moveTo(j, 2);
        ctxt.lineTo(j, 358);

        //draw horizontal lines
        ctxt.moveTo(2, j);
        ctxt.lineTo(358, j);
      }
      ctxt.lineWidth = this.lineWidth;
      ctxt.stroke();

      //draw the cursor
      if (!this.buildMode){
        this.drawRect(this.currentCursor[0], this.currentCursor[1], "#880017", 1)
      }
      
    },
    gameGridOnClick(row, col) {
      if (this.buildMode){
        if (this.currentCells.get(row+"-"+col)){
          this.currentCells.delete(row+"-"+col)
          this.drawRect(row, col, "white", 2)
        }else{
          this.currentCells.set(row+"-"+col, {row:row, col:col})
          this.drawRect(row, col, "#880017", 1)
        }
        // this.updateGrid(row, col, col+1)
      } else{
        console.log("app method invoked")
      } 
    },
    buildModeEvents(event_data){
      if (event_data.msg === 'toggle-mode'){
        this.toggleMode()
      }else if (event_data.msg === 'reset'){
        this.currentError = ""
        this.currentCells = new Map(),
        this.gamegrid = Array(6).fill().map(() => Array(6))
        this.reDrawGrid()
      }else if (event_data.msg === 'add-group'){
        if (event_data.val === -1){
          this.currentError = "please enter a positive integer"
          return
        }
        if (this.currentCells.size === 0){
          this.currentError = "please select the cells first"
          return
        }
        this.currentError = ""
        let min = {row:6, col:6}
        let mapIterator = this.currentCells.values();
        let i = 0;
        while (i < this.currentCells.size) {
          let val = mapIterator.next().value
          this.gamegrid[val.row][val.col] = event_data.grpId
          if ((min.row > val.row) || (min.row == val.row && min.col > val.col)){
            min.row = val.row
            min.col = val.col
          }
          i++;
        }

        this.drawGame()
        this.writeValAndOpAt(min, event_data.val, event_data.opp)

        mapIterator = this.currentCells.values();
        i = 0;
        while (i < this.currentCells.size) {
          let val = mapIterator.next().value
          this.drawRect(val.row, val.col, "white", 2)
          i++;
        }
        this.currentCells = new Map()
      }

    },
    solveModeEvents(event_data){
      if (event_data.msg === 'toggle-mode'){
        this.toggleMode()
      }else if (event_data.msg === 'update-grid'){
        this.updateGrid(event_data.row, event_data.col, event_data.data, event_data.reset)
      }else if (event_data.msg === 'update-cursor'){
        this.updateCursor(event_data.row, event_data.col)
      }

    },
    updateGrid(row, col, data, reset) {
      const gridCtxt = this.gridCanv.getContext("2d")
      const cellSize = this.cellSize;
      let pos_x = 0;
      let pos_y = 0;
      let center_x = 0;
      let center_y = 0;

      if (reset){
        for (let r = 0; r <= 5; r++){
          for(let c = 0; c <= 5; c++){
            pos_x = c * cellSize + 14;
            pos_y = r * cellSize + 14;

            gridCtxt.fillStyle = "white";
            gridCtxt.beginPath();
            gridCtxt.moveTo(pos_x, pos_y)
            gridCtxt.rect(pos_x, pos_y, 36, 36);
            gridCtxt.fill();
          }
        }
      }else{
        pos_x = col * cellSize + 14;
        pos_y = row * cellSize + 14;
        center_x = col * cellSize + 25;
        center_y = row * cellSize + 38;
        gridCtxt.fillStyle = "white";
        gridCtxt.beginPath();
        gridCtxt.moveTo(pos_x, pos_y)
        gridCtxt.rect(pos_x, pos_y, 36, 36);
        gridCtxt.fill();
        gridCtxt.font = "20px Arial";
        gridCtxt.fillStyle = 'black';
        gridCtxt.fillText(data, center_x, center_y)
      }
    },
    updateCursor(new_row, new_col){
      this.prevCursor = [this.currentCursor[0], this.currentCursor[1]]
      
      //deleting the rectangle at previous cursor
      this.drawRect(this.prevCursor[0], this.prevCursor[1], "white", 2)

      this.currentCursor = [new_row, new_col]

      //drawing a new rectangle at new cursor
      this.drawRect(this.currentCursor[0], this.currentCursor[1], "#880017", 1)
    },
    drawGame(){
      const lineWidth = this.lineWidth
      let ctxt = this.gridCanv.getContext('2d')
      ctxt.strokeStyle = "#00a876"

      let k_horizontal_start = 60;
      let k_vertical_start = 2;
      for (let i=0; i<6; i++){
        for (let j=0; j<6; j++){
          // drawing vertical line
          if (j<5 && this.gamegrid[i][j] != this.gamegrid[i][j+1]){
              ctxt.beginPath();
              ctxt.moveTo(k_horizontal_start, k_vertical_start);
              ctxt.lineTo(k_horizontal_start, k_vertical_start+60);
              ctxt.lineWidth = lineWidth*4;
              ctxt.stroke();
              
            }
          k_horizontal_start = k_horizontal_start + 60
        }
        k_horizontal_start = 60
        k_vertical_start = k_vertical_start + 60
      }

      k_horizontal_start = 2;
      k_vertical_start = 60;
      for (let i= 0; i < 6;i++){
        for (let j = 0; j<6; j++){
          // drawing horizontal line
          if (i < 5 && this.gamegrid[i][j] != this.gamegrid[i+1][j]){
              ctxt.beginPath();
              ctxt.moveTo(k_horizontal_start, k_vertical_start);
              ctxt.lineTo(k_horizontal_start+60, k_vertical_start);
              ctxt.lineWidth = lineWidth*4;
              ctxt.stroke();
            }
          k_horizontal_start = k_horizontal_start + 60
        }
        k_horizontal_start = 2
        k_vertical_start = k_vertical_start + 60
      }
    },
    drawRect(row, col, strokestyle, linewidth) {
      const cellSize = this.cellSize;
      const gridCtxt = this.gridCanv.getContext("2d")
      gridCtxt.strokeStyle = strokestyle;
      gridCtxt.lineWidth = linewidth;
      gridCtxt.beginPath();
      gridCtxt.moveTo(col*cellSize+13, row*cellSize+13);
      gridCtxt.rect(col*cellSize+13, row*cellSize+13, 40, 40);
      gridCtxt.stroke();
    },
    reDrawGrid(){
      let ctxt = this.gridCanv.getContext('2d')
      // whiting out the previous board
      ctxt.fillStyle = "white";
      ctxt.beginPath();
      ctxt.moveTo(0, 0)
      ctxt.rect(0, 0, 360, 360);
      ctxt.fill();

      //drawing the grid again
      this.drawGrid()

    },
    writeValAndOpAt(min, val, opp){
      const gridCtxt = this.gridCanv.getContext('2d')
      const cellSize = this.cellSize;
      let pos_x = 0;
      let pos_y = 0;
      pos_x = min.col * cellSize + 5;
      pos_y = min.row * cellSize + 13;
      gridCtxt.font = "12px Arial";
      gridCtxt.fillStyle = 'black';
      gridCtxt.beginPath();
      gridCtxt.moveTo(pos_x, pos_y);
      gridCtxt.fillText(`${val} ${opp}`, pos_x, pos_y)
      gridCtxt.stroke();
    },
  }
}
</script>

<style scoped>
  h3 {
    text-align: center;
    margin: 0;
    font-size: 1.25rem;
    color: #88005b;
  }
  .boardDisplay {
    width: 28%;
    margin: auto;
  }
</style>
