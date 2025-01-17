<template>
  <div class="palette--container">
    <div class="palette--colors">
      <div
        :class="{
          'palette--color-container': true,
          picked: drawingTool.currentColor === i - 1,
        }"
        v-for="i in 15"
        :key="i"
        @click="onColorClick($event, i - 1)"
        @mousemove="onColorMousemove($event, i - 1)"
      >
        <IconColorBlob class="palette--color" :color="paletteColors[i - 1]" />

        <div class="palette--selected-indicator"></div>
      </div>

      <div
        :class="{
          'palette--color-container': true,
          picked: drawingTool.currentColor === 15,
        }"
        @click="onColorClick($event, 15)"
        @mousemove="onColorMousemove($event, 15)"
      >
        <IconTransparentBlob class="palette--color" />
        <div class="palette--selected-indicator"></div>
      </div>
    </div>

    <div
      class="palette--button-hint left"
      @click="onColorClick($event, drawingTool.currentColor - 1)"
    >
      <div class="hint">L</div>
    </div>
    <div
      class="palette--button-hint right"
      @click="onColorClick($event, drawingTool.currentColor + 1)"
    >
      <div class="hint">R</div>
    </div>
  </div>
</template>

<script>
import DrawingTool from "~/libs/DrawingTool";

// svg icons
import IconColorBlob from "~/components/icons/IconColorBlob.vue";
import IconTransparentBlob from "~/components/icons/IconTransparentBlob.vue";

export default {
  name: "Palette",
  components: {
    IconColorBlob,
    IconTransparentBlob,
  },
  props: {
    drawingTool: {
      type: DrawingTool,
      required: true,
    },
  },
  data: function () {
    const paletteColors = [];
    for (let i = 0; i < 15; ++i)
      paletteColors.push(this.drawingTool.getPalette(i));
    return {
      paletteColors,
    };
  },
  methods: {
    invalidIdx: function (idx) {
      if (idx > 15 || idx < 0) {
        console.log("detected invalid current color value:", idx);
        return true;
      }
    },
    onColorClick: function (event, idx) {
      if (this.invalidIdx(idx)) return;
      // DOUBLE CLICK, OPEN COLOR PICKER
      if (this.drawingTool.currentColor === idx) {
        this.$emit("change-color-picker");
      }
      this.$emit("change-current-color", idx);
    },
    onColorMousemove: function (event, idx) {
      if (event.buttons === 1) {
        if (this.invalidIdx(idx)) return;
        this.$emit("change-current-color", idx);
      }
    },
    updatePaletteColors: function () {
      for (let i = 0; i < 15; ++i) {
        const paletteColor = this.drawingTool.getPalette(i);
        this.paletteColors.splice(i, 1, paletteColor);
      }
    },
  },
  mounted: function () {
    this.drawingTool.onColorChange(this.updatePaletteColors);
  },
  beforeDestroy: function () {
    this.drawingTool.onColorChangeRemove(this.updatePaletteColors);
  },
};
</script>

<style lang="scss" scoped>
@import "styles/colors";
@import "styles/transitions";
@import "styles/positioning";
@import "styles/screens";

.palette--container {
  display: inline-block;
  user-select: none;

  position: relative;
  top: 0;
  left: 0;
}

.palette--colors {
  box-sizing: border-box;
  padding: 5px 10px;

  display: grid;
  grid-template-columns: repeat(8, 20px);
  column-gap: 5px;
  row-gap: 15px;
  grid-template-rows: min-content;
  justify-content: center;
  justify-items: center;

  background-color: $pink-lace;
  border-radius: 30px;

  @include phone-landscape {
    grid-template-columns: repeat(8, 38px);
  }
  @include tablet-portrait {
    padding: 15px 30px 15px 30px;
    grid-template-columns: repeat(16, 30px);
    column-gap: 5px;
  }
  @include tablet-landscape {
    column-gap: 10px;
    grid-template-columns: repeat(16, 34px);
  }
  @include desktop {
    column-gap: 15px;
    grid-template-columns: repeat(16, 36px);
  }
}

.palette--color-container {
  @include relative-in-place;

  transition: transform 0.15s $energetic;
  display: inline-block;
  cursor: pointer;
  transform: scale(1);
  &:hover {
    transform: scale(1.2);
  }
  &:active {
    transition: transform 0.05s $energetic;
    transform: scale(1);
  }

  .palette--selected-indicator {
    width: 30px;
    height: 6px;

    position: absolute;
    left: 50%;
    bottom: -8px;
    transform: translate(-50%, 0px) scale(0);

    transition: transform 0.1s $energetic;
    background-color: $persian-green;
    border-radius: 4px;
    opacity: 0;
  }
  &.picked .palette--selected-indicator {
    transform: translate(-50%, 0px) scale(1);
    opacity: 1;
  }
}

.palette--color {
  width: 100%;
}

.palette--button-hint {
  position: absolute;
  background-color: $pink-lace;
  padding: 5px;
  box-sizing: content-box;
  bottom: 8px;
  $horizontal-correction: 10px;

  &.left {
    left: $horizontal-correction;
    transform: translate(-50%, 50%);
  }
  &.right {
    right: $horizontal-correction;
    transform: translate(50%, 50%);
  }

  // display: block;
  display: none;
  color: white;
  border-radius: 999px;

  width: 30px;
  height: 30px;

  .hint {
    width: 30px;
    height: 30px;
    font-size: 1.25rem;
    line-height: 30px;
    text-align: center;

    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);

    background-color: $olive-haze;
    border-radius: 999px;

    cursor: pointer;
  }
}
</style>
