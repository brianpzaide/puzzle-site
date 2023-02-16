<template>
    <section>
        <!-- <p><strong>Build Mode</strong></p> -->
        <div class="errorDisplay">
            <span v-if="currentError">{{currentError}}</span>
            <span v-else>&nbsp; </span>
        </div>

        <div class="valueDisplay">
            <label>Value: </label>
            <input type="number" size="9" v-model="currentValue"/>
        </div>
        <div class="valueDisplay">
            <label>Opp: {{currentOpp}}</label>
        </div>
        <div>
            <canvas id="oppscanvas" ref="oppscanvas" width="240" height="180"></canvas>
        </div>
  </section>
</template>

<script>
export default {
    props:[
        'currentError'
    ],
    data() {
        return{
            oppsCanv:null,
            displayError: false,
            currentGroupId: 1,
            currentValue: -1,
            currentOpp: "+",
            currentCells:[],
            oppsMap: [{opp:"+", adjustments: [0, 0]},{opp:"-", adjustments:[3, -1]}, {opp:"x", adjustments:[0, -1]},{opp:"/", adjustments:[4,0]}],
            lineWidth: 1,
            cellSize: 60,
            buttonSize: 50
        }
        
    },
    mounted(){
        this.oppsCanv = this.$refs.oppscanvas
        this.drawOppsGrid()
        this.oppsCanv.addEventListener('mousedown', (e) => {
            const cellSize = 60;
            const col = Math.floor(e.offsetX / cellSize);
            const row = Math.floor(e.offsetY / cellSize);
            this.oppsGridOnClick(row, col);  		  
        })
    },
    methods: {
        drawOppsGrid(){
            const ctxt = this.oppsCanv.getContext("2d")
            const lineWidth = this.lineWidth
            const cellSize = this.cellSize
            const buttonSize = this.buttonSize
            ctxt.strokeStyle = "black";
            //draw vertical lines
            ctxt.moveTo(2, 2);
            ctxt.lineTo(2, 178);

            ctxt.moveTo(238, 2);
            ctxt.lineTo(238, 178);

            //draw horizontal lines
            ctxt.moveTo(2, 2);
            ctxt.lineTo(238, 2);

            ctxt.moveTo(2, 178);
            ctxt.lineTo(238, 178);

            ctxt.lineWidth = lineWidth*2;
            ctxt.stroke();
            
            var pos_x = 0;
            var pos_y = 0;
            var center_x = 0;
            var center_y = 0;
            var k = 0;
            
            var row = 0
            for(let col = 0; col <= 3; col++){
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
                ctxt.fillText(this.oppsMap[k].opp, center_x+this.oppsMap[k].adjustments[0], center_y+this.oppsMap[k].adjustments[1])
                k++
            }

            var r = 1
            var c = 0
            pos_x = c * cellSize + 5;
            pos_y = r * cellSize + 5;
            center_x = c * cellSize + 40;
            center_y = r * cellSize + 35;
            ctxt.fillStyle = "#00a876";
            ctxt.beginPath();
            ctxt.rect(pos_x, pos_y, 2*cellSize-8, buttonSize-5);
            ctxt.fill();
            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("add", center_x, center_y)

            c = 2
            pos_x = c * cellSize + 5;
            pos_y = r * cellSize + 5;
            center_x = c * cellSize + 40;
            center_y = r * cellSize + 35;
            ctxt.fillStyle = "#00a876";
            ctxt.beginPath();
            ctxt.rect(pos_x, pos_y, 2*cellSize-10, buttonSize-5);
            ctxt.fill();
            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("reset", center_x, center_y)

            r = 2
            c = 0
            pos_x = c * cellSize + 5;
            pos_y = r * cellSize + 5;
            center_x = c * cellSize + 70;
            center_y = r * cellSize + 35;
            ctxt.fillStyle = "#88005b";
            ctxt.beginPath();
            ctxt.rect(pos_x, pos_y, 4*cellSize-10, buttonSize-5);
            ctxt.fill();
            ctxt.font = "20px Arial";
            ctxt.fillStyle = 'white';
            ctxt.fillText("start solving", center_x, center_y)
        },
        oppsGridOnClick(row, col){
            
            if (row == 0){
                this.currentOpp = this.oppsMap[col].opp
            }else if (row === 1){
                if (col <= 1 && this.currentValue === -1){
                    this.displayError = true
                    return
                }
                var msg = col <= 1 ? "add-group": "reset"
                var grpId = this.currentGroupId
                var val = this.currentValue
                var opp = this.currentOpp 
                this.$emit('buildmode-events', {msg:msg, grpId: grpId, val: val, opp: opp})
            }else{
                this.$emit('buildmode-events', {msg:"toggle-mode", grpId: "", val: -1, opp: ""})
            }

            
        },
    }
}
</script>

<style scoped>
    section {
        width: 20%;
        margin: auto;
    }

    .valueDisplay{
        font-family: 'Roboto mono',monospace;
        color: #00a876;
        font-size: 20px;
    }

    .errorDisplay{
        font-family: 'Roboto mono',monospace;
        color: #fb0909;
        font-size: 15px;
    }
</style>