<template>
    <section>
        <!-- <p><strong>Solve Mode</strong></p> -->
        <div id="timer" class="timerDisplay"><strong>{{stopwatch}}</strong></div>
        <canvas id="controllscanvas" ref="controllscanvas" width="360" height="180"></canvas>
  </section>
</template>

<script>
export default {
    props:[
        'currentCursor'
    ],
    data() {
        return{
            mins: 0,
            secs: 0,
            stopwatch: "00:00",
            stopwatchInterval: 1000,
            stopwatchStarted: false,
            buttonText: "done",
            timeOutId: null,
            ctrlCanv: null,
            lineWidth: 1,
            cellSize: 60,
            buttonSize: 52
        }
        
    },
    mounted(){
        console.log(this.currentCursor)
        this.ctrlCanv = this.$refs.controllscanvas
        this.drawctrlGrid()
        this.ctrlCanv.addEventListener('mousedown', (e) => {
            const col = Math.floor(e.offsetX / this.cellSize);
            const row = Math.floor(e.offsetY / this.cellSize);
            console.log("click (x, y)", e.offsetX, e.offsetY);
            console.log("click (row, col)", row, col);
            this.controlGridOnClick(row, col);  		  
        })
        this.resetStopwatch()
        this.startStopwatch()
    },
    beforeUnmount(){
        this.stopStopwatch()
    },
    methods:{
        drawctrlGrid(){
            const ctxt = this.ctrlCanv.getContext("2d");
            const lineWidth = this.lineWidth;
            //main
            ctxt.strokeStyle = "black";
            //draw vertical lines
            ctxt.moveTo(2, 2);
            ctxt.lineTo(2, 178);

            ctxt.moveTo(358, 2);
            ctxt.lineTo(358, 178);

            //draw horizontal lines
            ctxt.moveTo(2, 2);
            ctxt.lineTo(358, 2);

            ctxt.moveTo(2, 178);
            ctxt.lineTo(358, 178);

            ctxt.lineWidth = lineWidth*2;
            ctxt.stroke();

            const cellSize = this.cellSize;
            const buttonSize = this.buttonSize;
            var pos_x = 0;
            var pos_y = 0;
            var center_x = 0;
            var center_y = 0;
            var k = 1;
            
            for(let row = 0; row < 2; row++){
                for(let col = 0; col < 3; col++){
                    pos_x = col * cellSize + 5;
                    pos_y = row * cellSize + 5;
                    center_x = col * cellSize + 25;
                    center_y = row * cellSize + 38;
                    ctxt.fillStyle = "#00a876";
                    ctxt.beginPath();
                    ctxt.rect(pos_x, pos_y, buttonSize, buttonSize);
                    ctxt.fill();
                    ctxt.font = "20px Arial";
                    ctxt.fillStyle = 'white';
                    ctxt.fillText(k, center_x, center_y)
                    k++
                }
            }

            this.updateToggleButton()

            for(let row = 0; row < 3; row++){
                for(let col = 3; col < 6; col++){

                    if((row == 0 && col == 4) || (row == 1 && col == 3) || (row == 1 && col == 4) || (row == 1 && col == 5) || (row == 2 && col == 4)){
                        pos_x = col * cellSize + 5;
                        pos_y = row * cellSize + 5;
                        center_x = col * cellSize + 25;
                        center_y = row * cellSize + 38;
                        if (row == 1 && col == 4){
                        ctxt.fillStyle = "#88005b";
                        }else{
                        ctxt.fillStyle = "#00a876";
                        }
                        
                        ctxt.beginPath();
                        ctxt.rect(pos_x, pos_y, buttonSize, buttonSize);
                        ctxt.fill();
                    }   
                }
            }

            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("\u{1431}", 4 * cellSize + 24, 0 * cellSize + 37)

            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("\u{1438}", 3 * cellSize + 24, 1 * cellSize + 37)

            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("\u{1433}", 5 * cellSize + 24, 1 * cellSize + 37)

            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("\u{142F}", 4 * cellSize + 24, 2 * cellSize + 37)

            ctxt.font = "18px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("reset", 4 * cellSize + 11, 1 * cellSize + 36)
        },
        controlGridOnClick(row, col){
            var msg = ""
            var r = 0
            var c = 0
            var data = 0
            var reset = false
            if (row === 2 && col < 3){
                if (this.buttonText === "done"){
                    this.stopStopwatch()
                    this.buttonText = "back"
                    this.updateToggleButton()
                    return
                }else if(this.buttonText === "start"){
                    this.startStopwatch()
                    this.buttonText = "done"
                    this.updateToggleButton()
                    return
                }
                msg = "toggle-mode"
            }
            else if (col < 3){
               msg = "update-grid"
               r = this.currentCursor[0]
               c = this.currentCursor[1]
               data = row*3 + col + 1
               reset = false
            }
            else if (row === 1 && col === 4){
               msg = "update-grid"
               r = -1
               c = -1
               data = -1
               reset = true
               this.stopStopwatch()
               this.buttonText = "start"
               this.updateToggleButton()
            }
            else if (row === 0 && col === 4){
                if (this.currentCursor[0] > 0){
                    msg = "update-cursor"
                    r = this.currentCursor[0]-1
                    c = this.currentCursor[1]
                }else{
                    return
                }
            }
            if (row === 1 && col === 3){
                if (this.currentCursor[1] > 0){
                    msg = "update-cursor"
                    r = this.currentCursor[0]
                    c = this.currentCursor[1]-1
                }else{
                    return
                }
            }
            if (row === 1 && col === 5){
                if (this.currentCursor[1] < 5){
                    msg = "update-cursor"
                    r = this.currentCursor[0]
                    c = this.currentCursor[1]+1
                }else{
                    return
                }
            }
            if (row === 2 && col === 4){
                if (this.currentCursor[0] < 5){
                    msg = "update-cursor"
                    r = this.currentCursor[0]+1
                    c = this.currentCursor[1]
                }else{
                    return
                }
            }
            this.$emit("solvemode-events", {msg:msg, row:r,col:c,data:data,reset:reset})
        },
        updateToggleButton(){
            const ctxt = this.ctrlCanv.getContext("2d")
            const r = 2
            const c = 0
            const cellSize = this.cellSize
            const buttonSize = this.buttonSize
            const pos_x = c * cellSize + 5;
            const pos_y = r * cellSize + 5;
            ctxt.fillStyle = "#88005b";
            ctxt.beginPath();
            ctxt.rect(pos_x, pos_y, 3*cellSize-8, buttonSize-4);
            ctxt.fill();
            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText(this.buttonText, 66, 155)
        },
        startStopwatch(){
            this.timeOutId = setTimeout(this.updateStopwatch, this.stopwatchInterval);
            this.stopwatchStarted = true
        },
        updateStopwatch(){
            this.secs++
            if (this.secs === 60){
                this.mins++
                this.secs = 0
            }
            let mins = `${this.mins}`
            let secs = `${this.secs}`
            if (this.mins < 10){
                mins = '0'+mins
            }
            if (this.secs < 10){
                secs = '0'+secs
            }
            this.stopwatch = `${mins}:${secs}`
            this.timeOutId = setTimeout(this.updateStopwatch, this.stopwatchInterval)
        },
        stopStopwatch(){
            if (this.timeOutId){
                clearTimeout(this.timeOutId)
            }
        },
        resetStopwatch(){
            this.mins = 0
            this.secs = 0
            this.stopwatch = `00:00`
        }
    }
}
</script>

<style scoped>
    section {
        width: 28%;
        margin: auto;
    }

    .timerDisplay{
        font-family: 'Roboto mono',monospace;
        color: #00a876;
        font-size: 30px;
    }
</style>