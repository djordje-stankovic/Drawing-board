<template>
  <div>
    <div
      class="row"
      style="margin-left:0"
      :width="this.canvasSetingsMerged.width"
      :height="this.canvasSetingsMerged.height"
    >
      <canvas
        :class="styleCursor"
        v-bind:style="{border: borderStyle }"
        :width="this.canvasSetingsMerged.width"
        :height="this.canvasSetingsMerged.height"
        @mousedown="startPainting"
        @mouseup="finishedPainting"
        @mousemove="draw"
        id="canvas"
      ></canvas>
      <b-button-group :style="styleCursorReverse">
        <b-button title="Delete" @click="resetCanvasAndEmptyList">
          <b-icon
            v-bind:style="{margin:colorMargin, width:colorWidth,height: colorHeight}"
            icon="trash-fill"
            aria-hidden="true"
          ></b-icon>
        </b-button>
      </b-button-group>
      <div v-bind:class="SideBar" :height="this.canvasSetingsMerged.height">
        <div>
          <b-button-group class="mr-1">
            <b-button style="pading:0" title="Undo">
              <b-icon
                disabled
                icon="arrow-bar-left"
                @click="loadPreviousCanvasImage"
                aria-hidden="true"
              ></b-icon>
            </b-button>
            <b-button style="pading:0" title="Redo">
              <b-icon icon="arrow-bar-right" @click="loadNextCanvasImage" aria-hidden="true"></b-icon>
            </b-button>
          </b-button-group>
        </div>
        <div>
          <b-button-group class="mr-1">
            <b-button title="Squares">
              <b-icon icon="square" @click="changeStyleOfDrawing('square')" aria-hidden="true"></b-icon>
            </b-button>
            <b-button title="Pancil">
              <b-icon icon="pencil" @click="changeStyleOfDrawing('round')" aria-hidden="true"></b-icon>
            </b-button>
          </b-button-group>
        </div>
        <div>
          <b-button-group class="mr-1">
            <b-button title="Eraze">
              <b-icon icon="layout-sidebar" @click="changeToErazer" aria-hidden="true"></b-icon>
            </b-button>
          </b-button-group>
        </div>
        <div>
          <b-button-group class="mr-1">
            <b-button title="Delete">
              <b-icon icon="trash-fill" @click="resetCanvasAndEmptyList" aria-hidden="true"></b-icon>
            </b-button>
          </b-button-group>
        </div>
        <input id="sizeOfDrawing" @change="changeSizeOfDrawing" style="width : 50px" type="number" />
      </div>
    </div>
    <div
      class="flex-gap"
      v-bind:style="{background :colorSettings.background}"
      style="margin-top : 0"
    >
      <div
        v-for="color in colors"
        v-bind:key="colors[color]"
        v-bind:style="{background:color, margin:colorMargin, width:colorWidth,height: colorHeight}"
        @click="changeColor(color)"
      >&nbsp;</div>
      <input @change="changeColorFromInput()" id="color" type="color" />
    </div>
  </div>
</template>


<script>
export default {
  props: {
    //Default value for Canvas Seting
    canvasSetings: {
      type: Object,
      required: false,
      default: () => ({}),
    },
    //Default array of colors for fast color picker
    colors: {
      type: Array,
      default: function () {
        return [
          "#FC2C00",
          "#5FFC00",
          "#00FCCE",
          "#0400FC",
          "#FC00F8",
          "#FC006B",
          "#FCBB00",
          "#00FC63",
          "#008DFC",
        ];
      },
    },
    //SideBar setings
    sideBar: {
      default: true,
    },
    //Default color setings
    colorSettings: {
      type: Object,
      required: false,
      default: () => ({}),
    },
  },
  //Make Sure that if user don't pass all props they get fill with default values.
  beforeMount() {
    Object.assign(
      this.canvasSetingsMerged,
      this.canvasSetingsDefault,
      this.canvasSetings
    );
    Object.assign(
      this.colorSetingMerged,
      this.coolorSetingsDefault,
      this.colorSettings
    );
  },
  mounted() {
    document.getElementById("sizeOfDrawing").value = this.sizeOfDrowing;
    this.canvas = document.getElementById("canvas");

    this.ctx = canvas.getContext("2d");
    this.color = document.getElementById("color");
    this.ctx.strokeStyle = "#e1ff00";
  },
  data() {
    return {
      drawing: true,
      canvasListFull: false,
      eraze: false,
      sizeOfDrowing: 4,
      styleOfDrawing: "round",
      positionInCanvasList: 0,
      canvasListForUndo: [],
      canvas: null,
      ctx: null,
      color: "red",
      //Merged object for borded Setings from props and default value
      canvasSetingsMerged: {},
      //Merged object for color Setings from props and default value
      colorSetingMerged: {},

      //Default Canvas Setings prop values
      canvasSetingsDefault: {
        width: 300,
        height: 300,
        border: {
          color: "black",
          style: "solid ",
          size: 2,
        },
      },
      //Default Border Setings
      borderDefault: {
        color: "black",
        style: "solid ",
        size: 2,
      },
      //Default Color Seting prop values
      coolorSetingsDefault: {
        margin: 2,
        colorWidth: 15,
        colorHeight: 15,
        background: "White",
      },
    };
  },
  computed: {
    styleCursorReverse: function () {
      if (this.sideBar == true) return "display: none;";
    },
    //Show side bar with tools if user want
    SideBar: function () {
      return this.sideBar ? "flex-colum" : "hiden";
    },
    //Change canvas cursor based on user action

    styleCursor: function () {
      return this.drawing ? "pen" : "erser";
    },
    //Seting canvas border
    borderStyle: function () {
      return `${this.canvasSetingsMerged.border.size}px ${this.canvasSetingsMerged.border.style} ${this.canvasSetingsMerged.border.color}`;
    },
    //Seting size of Color for picking
    colorWidth: function () {
      return `${this.colorSetingMerged.colorWidth}px`;
    },
    //Seting size of Color for picking
    colorHeight: function () {
      return `${this.colorSetingMerged.colorHeight}px`;
    },
    //Seting size of Color for picking
    colorMargin: function () {
      return `${this.colorSetingMerged.margin}px`;
    },
  },
  methods: {
    changeToErazer() {
      this.eraze = true;
      this.drawing = false;
    },
    changeSizeOfDrawing() {
      let size = document.getElementById("sizeOfDrawing").value;
      this.sizeOfDrowing = size;
      this.ctx.strokeStyle = this.color.value;
    },
    //Change style based on select button from menu and change cursor stile
    changeStyleOfDrawing(style) {
      this.styleOfDrawing = style;
      this.eraze = false;
      this.drawing = true;
      this.ctx.strokeStyle = this.color.value;
      console.log(this.styleOfDrawing);
    },

    // Take value of  color from palete
    saveCanvasInListForUndo() {
      if (this.canvasListForUndo.length < 19) {
        this.positionInCanvasList += 1;
        this.canvasListForUndo.push(
          document.getElementById("canvas").toDataURL()
        );
      } else {
        this.canvasListForUndo.shift();
        this.canvasListForUndo.push(
          document.getElementById("canvas").toDataURL()
        );
      }
    },
    //Method which take previus picture of canvas and display  as undo

    loadPreviousCanvasImage() {
      if (this.positionInCanvasList > -1) {
        if (this.positionInCanvasList == 1) {
          this.resetCanvas();
          return;
        } else {
          --this.positionInCanvasList;

          var canvasPic = new Image();
          canvasPic.src = this.canvasListForUndo[this.positionInCanvasList - 1];
          canvasPic.onload = () => {
            this.resetCanvas();
            this.ctx.drawImage(canvasPic, 0, 0);
          };
        }
      }
    },
    //Method for Redo
    loadNextCanvasImage() {
      if (this.positionInCanvasList > this.canvasListForUndo.length) {
        return;
      } else {
        var canvasPic = new Image();
        canvasPic.src = this.canvasListForUndo[this.positionInCanvasList - 1];

        canvasPic.onload = () => {
          this.resetCanvas();
          this.ctx.drawImage(canvasPic, 0, 0);
        };
      }
      if (this.positionInCanvasList == this.positionInCanvasList - 1) {
        return;
      } else {
        ++this.positionInCanvasList;
      }
    },
    changeColorFromInput() {
      this.ctx.strokeStyle = document.getElementById("color").value;
      this.drawing = true;
    },
    draw(e) {
      if (!this.painting) return;
      this.ctx.strokeStyle = this.color.value;
      if (this.eraze == true) {
        this.ctx.strokeStyle = "white";
      }

      this.ctx.lineCap = this.styleOfDrawing;
      this.ctx.lineWidth = this.sizeOfDrowing;
      let rect = this.canvas.getBoundingClientRect();
      let x = e.clientX - rect.left;
      let y = e.clientY - rect.top;

      this.ctx.lineTo(x, y);
      this.ctx.stroke();

      this.ctx.beginPath();
      this.ctx.moveTo(x, y);
    },

    startPainting(e) {
      this.ctx.strokeStyle = this.color.value;
      this.painting = true;

      this.draw(e);
    },
    finishedPainting() {
      this.saveCanvasInListForUndo();
      this.painting = false;
      this.ctx.beginPath();
    },
    resetCanvasAndEmptyList() {
      this.resetCanvas();
      this.canvasListForUndo = [];
      this.positionInCanvasList = 0;
    },
    resetCanvas() {
      this.canvas = document.getElementById("canvas");
      this.canvas.width = this.canvasSetingsDefault.width;
      this.ctx.strokeStyle = this.color.value;
    },
    changeColor(color) {
      this.color.value = color;
      this.ctx.strokeStyle = color;
      document.getElementById("color").value = color;
      this.eraze = false;
      this.drawing = true;
    },
  },
};
</script>

<style>
.erser {
  cursor: url("http://www.rw-designer.com/cursor-extern.php?id=85157"), auto;
}
.pen {
  cursor: url("http://www.rw-designer.com/cursor-extern.php?id=41296"), auto;
}
.hiden {
  display: none;
}
.container {
  display: inline-flex;
  flex-direction: row;
  margin: 0px !important;
  padding: 0px !important;
  flex-wrap: wrap;
}
.btn {
  padding: 0.1rem !important;
}
.mr-1 {
  padding-top: 1px !important;
}
.flex-colum {
  display: flex;
  flex-direction: column;
}
.flex-gap {
  display: inline-flex;
  flex-wrap: wrap;
}
.flex-gap > div {
  margin: 1px;
  margin: 0, 0 0, 0;
}
div:empty {
  width: 10px;
  height: 10px;
}
</style>
