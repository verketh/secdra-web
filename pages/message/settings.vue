<template>
  <div class="card settings-box">
    <p><strong>消息通知开关</strong></p>
    <div class="row">
      <div class="col-10">评论我的</div>
      <div class="col-20">
        <Checkbox
          v-model="settingsForm.commentStatus"
          is-switch
          color="primary"
        ></Checkbox>
      </div>
    </div>
    <div class="row">
      <div class="col-10">回复我的</div>
      <div class="col-20">
        <Checkbox
          v-model="settingsForm.replyStatus"
          is-switch
          color="primary"
        ></Checkbox>
      </div>
    </div>
    <div class="row">
      <div class="col-10">关注我的</div>
      <div class="col-20">
        <Checkbox
          v-model="settingsForm.followStatus"
          is-switch
          color="primary"
        ></Checkbox>
      </div>
    </div>
  </div>
</template>
<script>
import { mapActions } from "vuex"

export default {
  watch: {
    settingsForm: {
      handler(val) {
        this.saveSetting(val)
      },
      deep: true
    }
  },
  async asyncData({ store, req, redirect, route, $axios }) {
    store.commit("message/MChangeType", { type: "settings", reset: false })
    const { data: result } = await $axios.get(`/message/getSettings`)
    const settingsForm = {
      id: result.data.id,
      commentStatus: result.data.commentStatus,
      followStatus: result.data.followStatus,
      replyStatus: result.data.replyStatus
    }
    return {
      settingsForm
    }
  },
  methods: {
    ...mapActions("message", ["ASaveSetting"]),
    async saveSetting(val) {
      await this.ASaveSetting(val)
    }
  }
}
</script>

<style type="text/less" lang="less" scoped>
@import "../../assets/style/color";
@import "../../assets/style/config";
@import "../../assets/style/mixin";

.settings-box {
  text-align: left;
  padding: 10px 30px;
  line-height: 35px;
  box-shadow: none;
  .row {
    width: 400px;
  }
}
</style>
