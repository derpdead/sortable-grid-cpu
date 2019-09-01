<template>
    <div :class="['column', { 'column--dragged': draggedIndex === index}]" v-text="column"
         draggable
         @dragstart="onDragStart"
         @dragend="onDragEnd"
         @dragover="onDragOver">
    </div>
</template>

<script>
export default {
  name: 'GridColumn',
  props: {
    index: {
      type: Number,
      required: true,
    },
    draggedIndex: {
      type: Number,
      default: -1,
    },
    column: {
      type: Number,
      required: true,
    },
  },
  methods: {
    onDragStart(event) {
      this.createColumnElClone(event);
      this.$emit('draggedIndex', this.index);
    },
    onDragEnd() {
      this.removeColumnElClone();
      this.$emit('draggedIndex', -1);
    },
    onDragOver(event) {
      const { clientX } = event;
      const isBefore = this.determinateDraggedColumnPositionState(clientX);

      if (this.draggedIndex === this.index
        || (isBefore && this.draggedIndex === this.index - 1)
        || (!isBefore && this.draggedIndex === this.index + 1)) {
        return false;
      }

      this.$emit('swapColumns', { from: this.index, to: this.draggedIndex });
      this.$emit('draggedIndex', this.index);

      return true;
    },
    createColumnElClone(event) {
      const { target } = event;
      const { width, height } = target.getBoundingClientRect();
      const clonedDOMElement = target.cloneNode(true);
      const clonedDOMElementStyle = `
        position: absolute;
        background-color: cadetblue;
        height: ${height}px;
        width: ${width}px;
      `;

      clonedDOMElement.setAttribute('style', clonedDOMElementStyle);
      clonedDOMElement.classList.add('cloned-column-element');
      document.body.appendChild(clonedDOMElement);
      event.dataTransfer.setDragImage(clonedDOMElement, clonedDOMElement.offsetWidth / 2, 0);
    },
    removeColumnElClone() {
      const clonedDOMElement = document.documentElement.querySelector('.cloned-column-element');
      document.body.removeChild(clonedDOMElement);
    },
    determinateDraggedColumnPositionState(clientX) {
      const { x: columnXPos, width: columnWidth } = this.$el.getBoundingClientRect();
      const normalizedHalfWidthFactor = 0.5;

      return (clientX - columnXPos) / columnWidth < normalizedHalfWidthFactor;
    },
  },
};
</script>

<style lang="scss" scoped>
    .column {
        display: flex;
        align-items: center;
        justify-content: center;
        background-color: cadetblue;
        color: white;

        &--dragged {
            background-color: cornflowerblue;
        }
    }
</style>
