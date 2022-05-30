<template>
  <button @click="addBox">add Box</button>
  <button @click="removeBox">remove Box</button>
  <v-stage ref="stage" :config="stageSize" @mousedown="handleStageMouseDown">
    <v-layer ref="layer">
      <v-rect v-for="item in rectangles" :key="item.id" :config="item" @transformend="handleTransformEnd"
        @dragend="handleDragEnd" />
      <v-transformer ref="transformer" :config="transformerConfig" />
    </v-layer>
  </v-stage>
  <div v-for="rectangle in rectangles">
    <code>{{ rectangle }}</code>
  </div>
</template>

<script>
import Konva from 'konva';
const width = window.innerWidth;
const height = 300;

export default {
  data() {
    return {
      stageSize: {
        width: width,
        height: height,
      },
      transformerConfig: {
        rotateEnabled: false,
        flipEnabled: false,
        // enabledAnchors: ['top-left', 'top-right', 'bottom-left', 'bottom-right']
        enabledAnchors: ['top-center', 'middle-right', 'bottom-center', 'middle-left'],
        ignoreStroke: true,
      },
      rectangles: [
        {
          id: "0",
          x: 10,
          y: 10,
          width: 100,
          height: 30,
          scaleX: 1,
          scaleY: 1,
          stroke: "rgb(0, 0, 0)",
          strokeWeight: 1,
          strokeScaleEnabled: false,
          name: 'rect1',
          draggable: true,
        },
        {
          id: "1",
          x: 150,
          y: 150,
          width: 100,
          height: 30,
          scaleX: 1,
          scaleY: 1,
          stroke: "rgb(0, 0, 0)",
          strokeWeight: 1,
          strokeScaleEnabled: false,
          name: 'rect2',
          draggable: true,
        },
      ],
      selectedShapeName: '',
    };
  },
  methods: {
    handleTransformEnd(e) {
      // shape is transformed, let us save new attrs back to the node
      // find element in our state
      const rect = this.rectangles.find(
        (r) => r.name === this.selectedShapeName
      );

      // update the state
      rect.x = e.target.x();
      rect.y = e.target.y();
      // console.log(rect.width, e.target.scaleX(), rect.width * e.target.scaleX())
      // console.log(rect.height, e.target.scaleY(), rect.height * e.target.scaleY())

      rect.scaleX = e.target.scaleX();
      rect.scaleY = e.target.scaleY();

      // rect.width = 100 * e.target.scaleX();
      // rect.height = rect.height * e.target.scaleY();

      // rect.scaleX = 1;
      // rect.scaleY = 1;

      // rect.stroke = "red"
    },
    handleStageMouseDown(e) {
      // filter native events
      if (!e.evt) {
        return;
      }
      // clicked on stage - clear selection
      if (e.target === e.target.getStage()) {
        this.selectedShapeName = '';
        this.updateTransformer();
        return;
      }

      // clicked on transformer - do nothing
      const clickedOnTransformer =
        e.target.getParent().className === 'Transformer';
      if (clickedOnTransformer) {
        return;
      }

      // find clicked rect by its name
      const name = e.target.name();
      const rect = this.rectangles.find((r) => r.name === name);
      if (rect) {
        this.selectedShapeName = name;
      } else {
        this.selectedShapeName = '';
      }
      this.updateTransformer();
    },
    updateTransformer() {
      // here we need to manually attach or detach Transformer node
      const transformerNode = this.$refs.transformer.getNode();
      const stage = transformerNode.getStage();
      const { selectedShapeName } = this;

      // console.log('here', typeof selectedShapeName, selectedShapeName)

      const selectedNode = stage.findOne('.' + selectedShapeName);
      // do nothing if selected node is already attached
      if (selectedNode === transformerNode.node()) {
        return;
      }

      if (selectedNode) {
        // attach to another node
        transformerNode.nodes([selectedNode]);
      } else {
        // remove transformer
        transformerNode.nodes([]);
      }
    },
    handleDragEnd(e) {
      const id = e.target.id()
      this.rectangles[id].x = e.target.x()
      this.rectangles[id].y = e.target.y()
    },
    addBox() {
      this.rectangles.push({
        id: `${this.rectangles.length}`,
        x: Math.random(0) * (width - 100),
        y: Math.random(0) * (height - 30),
        width: 100,
        height: 30,
        scaleX: 1,
        scaleY: 1,
        stroke: "rgb(0, 0, 0)",
        strokeWeight: 1,
        strokeScaleEnabled: false,
        name: `rect${this.rectangles.length + 1}`,
        draggable: true,
      })
    },
    removeBox() {
      this.rectangles.pop()
    },
  },
};
</script>
