<template>
  <transition
    name="fade"
    enter-active-class="fadeIn mask-duration"
    leave-active-class="fadeOut mask-duration"
  >
    <div v-show="visible" class="mask">
      <transition
        name="zoom"
        enter-active-class="zoomIn duration"
        leave-active-class="zoomOut duration"
        @after-leave="destroyElement"
      >
        <div
          v-show="visible"
          class="card"
          :class="{
            'dialog-persistent-animate': persistentAnimate,
            duration: persistentAnimate
          }"
          @animationend="persistentAnimationend()"
          @click.stop="(_) => {}"
        >
          <h3>
            {{ title }}
          </h3>
          <Field
            v-model="input"
            block
            color="primary"
            :placeholder="message"
          ></Field>
          <div class="btn-group">
            <Btn flat color="primary" @click="close">
              {{ noDesc }}
            </Btn>
            <Btn flat color="primary" @click="ok">
              {{ okDesc }}
            </Btn>
          </div>
        </div>
      </transition>
    </div>
  </transition>
</template>

<script>
import dialogMixin from "../../../assets/script/mixin/dialogMixin"
import { on, addClass } from "../../../assets/script/util/domUtil"

export default {
  componentName: "Prompt",
  mixins: [dialogMixin],

  data() {
    return {
      visible: false,
      input: "",
      closed: false,
      title: "提示",
      okDesc: `确定`,
      noDesc: `取消`,
      checkList: [],
      okCallback() {},
      noCallback() {}
    }
  },
  mounted() {
    on(document, "keydown", this.onEsc)
    addClass(document.body, "not-scroll")
  },
  methods: {
    ok() {
      this.closed = true
      this.okCallback && this.okCallback(this)
    },
    close() {
      this.closed = true
      this.noCallback && this.noCallback(this)
    }
  }
}
</script>

<style scoped lang="less" type="text/less">
@import "../../../assets/style/color";
@import "../../../assets/style/config";

.card {
  width: 450px;
  margin: 0 auto;
  vertical-align: middle;
  display: inline-block;
  padding: 15px;
  h3 {
    text-align: left;
    line-height: 40px;
  }
  input {
    margin: 20px 0;
  }
  .btn-group {
    margin-top: 10px;
    text-align: right;
  }
}
</style>
