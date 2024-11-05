<template>
  <div class="drag-zoom" ref="dragZoom" @mousedown="onMouseDown" @mousewheel="onMouseWheel">
    <div class="inner" ref="inner">
      <slot />
    </div>
  </div>
</template>

<script>
let isDragging = false;
let startX, startY;
let offsetX = 0,
  offsetY = 0;
let scale = 1;

export default {
  name: "DragZoom",
  mounted() {
    window.addEventListener("mousemove", this.onMouseMove);
    window.addEventListener("mouseup", this.onMouseUp);
  },
  methods: {
    onMouseDown(e) {
      isDragging = true;
      startX = e.clientX - offsetX;
      startY = e.clientY - offsetY;
    },
    onMouseWheel(e) {
      e.preventDefault();
      const scaleAmount = 0.2;
      const parentRect = this.$refs.dragZoom.getBoundingClientRect();
      const mouseX = e.clientX - parentRect.left;
      const mouseY = e.clientY - parentRect.top;

      // 缩放前鼠标相对于 childDiv 的位置
      const preZoomMouseX = (mouseX - offsetX) / scale;
      const preZoomMouseY = (mouseY - offsetY) / scale;

      // 更新缩放比例
      if (e.deltaY < 0) {
        scale += scaleAmount; // 放大
      } else {
        scale -= scaleAmount; // 缩小
      }
      scale = Math.min(Math.max(scale, 0.5), 6); // 限制缩放范围在 0.5 到 3 之间

      // 缩放后鼠标相对于 childDiv 的新位置
      const postZoomMouseX = preZoomMouseX * scale;
      const postZoomMouseY = preZoomMouseY * scale;

      // 计算并应用新的偏移量
      offsetX += mouseX - (postZoomMouseX + offsetX);
      offsetY += mouseY - (postZoomMouseY + offsetY);

      // 更新 childDiv 的 transform 样式
      this.$refs.inner.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${scale})`;
    },
    onMouseMove(e) {
      if (isDragging) {
        offsetX = e.clientX - startX;
        offsetY = e.clientY - startY;
        this.$refs.inner.style.transform = `translate(${offsetX}px, ${offsetY}px) scale(${scale})`;
      }
    },
    onMouseUp(e) {
      isDragging = false;
    },
  },
};
</script>

<style lang="less" scoped>
.drag-zoom {
  position: relative;
  overflow: hidden;
  .inner {
    position: absolute;
    width: 100%;
    height: 100%;
    transform-origin: left top;
  }
}
</style>
