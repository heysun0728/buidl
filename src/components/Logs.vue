<template>
  <div
    ref="logs"
    class="logs"
    v-html="'<p>' + logs.join('</p><p>') + '</p>'"
  ></div>
</template>

<script lang="ts">
import { Component, Vue, Watch } from "vue-property-decorator";
@Component({
  components: {},
  props: {
    type: String
  }
})
export default class Logs extends Vue {
  private logs: string[] | undefined;
  private type: string | undefined;

  public constructor() {
    super();
    if (this.type === "Lity") {
      this.logs = this.$store.state.outputs.lityLogs;
    } else {
      this.logs = this.$store.state.outputs.dappLogs;
    }
  }

  @Watch("logs")
  scrollToBottom() {
    this.$nextTick().then(() => {
      const elem = this.$refs.logs as any;
      elem.scrollIntoView(false);
    });
  }
}
</script>

<style lang="stylus">
@import "../assets/themes/light.styl"

.logs
  color $color
p
  margin 0 0 1em
  word-break break-all
  line-height 1.4
  .error
    color red
  .success
    color green
</style>

<style lang="stylus">
@import "../assets/themes/dark.styl"
body.dark-theme
  .logs
    color $color
</style>
