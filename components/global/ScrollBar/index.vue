<template>
  <div
    v-if="height < 100"
    class="scroll-bar-box"
    :style="{ position }"
    @click="scrollTo"
    @mouseenter="showScrollBar"
    @mouseleave="hideScrollBar"
  >
    <transition
      name="fade"
      enter-active-class="fadeIn duration"
      leave-active-class="fadeOut duration"
    >
      <div
        class="scroll-bar"
        :class="{ active: scrollBarActive }"
        :style="{
          height: height + `%`,
          transform: `translateY(${coordinate}%)`
        }"
        @mousedown.stop="scrollStart"
        @click.stop="() => {}"
      ></div>
    </transition>
  </div>
</template>

<script>
import windowMixin from "../../../assets/script/mixin/windowMixin"
import { off, on } from "../../../assets/script/util/domUtil"

export default {
  mixins: [windowMixin],
  data() {
    return {
      scrollElement: null,
      scrollHeight: 0,
      clientHeight: 0,
      startY: 0,
      startScrollTop: 0,
      mutationObserverFrameTick: false,
      scrollIngFrameTick: false,
      mutationObserver: null,
      resizeObserver: null,
      scrollBarStatusTimeout: null,
      scrollBarStatus: false,
      scrollBarActive: false
    }
  },
  computed: {
    position() {
      return this.scrollElement ? "absolute" : "fixed"
    },
    height() {
      return (this.clientHeight / this.scrollHeight) * 100
    },
    coordinate() {
      return (
        (((this.scrollTop / this.scrollHeight) * this.scrollHeight) /
          this.clientHeight) *
        100
      )
    }
  },
  mounted() {
    this.getHeight()
    const handle = () => {
      if (!this.mutationObserverFrameTick) {
        requestAnimationFrame(() => {
          this.getHeight()
          this.mutationObserverFrameTick = false
        })
        this.mutationObserverFrameTick = true
      }
    }
    this.resizeObserver = new ResizeObserver(() => {
      handle()
    })
    this.mutationObserver = new MutationObserver((mutations, _observer) => {
      handle()
    })
    if (this.scrollElement) {
      this.resizeObserver.observe(this.scrollElement)
      this.mutationObserver.observe(this.scrollElement, {
        attributes: true,
        characterData: true,
        childList: true,
        subtree: true
      })
    } else {
      this.resizeObserver.observe(document.documentElement)
    }
    on(document, "mouseup", this.scrollEnd)
  },
  beforeDestroy() {
    try {
      this.mutationObserver.disconnect()
      this.resizeObserver.disconnect()
    } catch (e) {}

    off(document, "mouseup", this.scrollEnd)
    off(document, "mousemove", this.scrollIng)
  },
  methods: {
    scrollStart(event) {
      this.startY = event.clientY
      this.startScrollTop = this.scrollTop
      this.scrollBarActive = true
      on(document, "mousemove", this.scrollIng)
    },
    scrollEnd() {
      off(document, "mousemove", this.scrollIng)
      // 必须等待一帧后再重置
      requestAnimationFrame(() => {
        this.startY = 0
        this.startScrollTop = 0
        this.scrollBarActive = false
      })
    },
    scrollIng(event) {
      if (!this.scrollIngFrameTick) {
        requestAnimationFrame(() => {
          const el = this.scrollElement || document.documentElement
          el.scrollTo(
            0,
            ((event.clientY - this.startY) / this.clientHeight) *
              this.scrollHeight +
              this.startScrollTop
          )
          this.scrollIngFrameTick = false
        })
        this.scrollIngFrameTick = true
      }
    },
    getHeight() {
      const el = this.scrollElement || document.documentElement
      this.scrollHeight = el ? el.scrollHeight : 0
      this.clientHeight = el ? el.clientHeight : 0
    },
    scrollTo(event) {
      const el = this.scrollElement || document.documentElement
      el.scrollTo({
        top:
          ((event.offsetY / this.clientHeight) * this.scrollHeight) /
            this.scrollTop >
          1
            ? this.scrollTop + this.clientHeight
            : this.scrollTop - this.clientHeight,
        behavior: "smooth"
      })
    },
    showScrollBar() {
      this.scrollBarStatus = true
    },
    hideScrollBar() {
      this.scrollBarStatus = false
    }
  }
}
</script>
<style type="text/less" lang="less" scoped>
@import "../../../assets/style/color";
@import "../../../assets/style/config";
@import "../../../assets/style/mixin";

.scroll-bar-box {
  top: 0;
  right: 0;
  height: 100%;
  /*bottom: 0;*/
  width: 6px;
  z-index: @mask-index - 2;
  .scroll-bar {
    user-select: none;
    touch-action: none;
    position: relative;
    border-radius: 10px;
    background-color: @font-color-dark-disabled;
    cursor: pointer;
    display: block;
    transition: background-color @default-animate-time;
    &.active,
    &:hover {
      background-color: @font-color-dark-fade;
    }
  }
}
</style>
