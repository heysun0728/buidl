<template>
  <div class="wallet">
    <div class="sig-actions">
      <h3>Accounts</h3>
      <button @click="newSig" title="New Account">+</button>
    </div>
    <ul>
      <li class="sig-item" v-for="(sig, index) in allSigs" :key="sig.address">
        <i
          v-if="defaultSig && defaultSig.address !== sig.address"
          @click="removeSig(index)"
          title="Remove"
          >&times;</i
        >
        {{ sig.address }}
        <input :ref="`sig${sig.address}`" type="hidden" :value="sig.address" />
        <div class="sig-item-op">
          <span v-if="defaultSig && defaultSig.address === sig.address"
            >Default</span
          >
          <a v-else @click="setDefault(sig)">Set Default</a>
          <a @click="copy(sig.address, $event)">Copy</a>
        </div>
      </li>
    </ul>
  </div>
</template>

<script lang="ts">
import { Component, Vue } from "vue-property-decorator";
import store from "../store";
import { Signature } from "../modules/wallet";
const EthUtil = require("ethereumjs-util");
const Secp256k1 = require("secp256k1");
const Crypto = require("crypto");

function newSig(): Signature {
  let privKey;
  do {
    privKey = Crypto.randomBytes(32);
  } while (!Secp256k1.privateKeyVerify(privKey));

  const address = EthUtil.privateToAddress(privKey);
  return new Signature(
    EthUtil.bufferToHex(address),
    EthUtil.bufferToHex(privKey)
  );
}
// init
if ((store.state as any).wallet.all.length === 0) {
  for (let i = 0; i < 5; i++) {
    let sig = newSig();
    store.dispatch("wallet/addSig", sig);
    if (i === 0) {
      store.dispatch("wallet/setDefault", sig);
    }
  }
}

@Component({
  components: {}
})
export default class Wallet extends Vue {
  get allSigs() {
    return this.$store.state.wallet.all;
  }

  get defaultSig() {
    return this.$store.state.wallet.default;
  }

  newSig() {
    let sig = newSig();
    this.$store.dispatch("wallet/addSig", sig);
  }

  removeSig(index: number) {
    this.$store.dispatch("wallet/removeSig", index);
  }

  setDefault(sig: Signature) {
    this.$store.dispatch("wallet/setDefault", sig);
  }

  copy(addr: string, e: any) {
    const input = (this.$refs[`sig${addr}`] as any)[0];
    input.setAttribute("type", "text");
    input.select();
    document.execCommand("copy");
    input.setAttribute("type", "hidden");
    e.target.innerText = "Copied";
    setTimeout(() => {
      e.target.innerText = "Copy";
    }, 500);
  }
}
</script>

<style lang="stylus">
@import "../assets/themes/light.styl"
.wallet
  display flex
  flex-direction column
  height 100%
  .sig-actions
    padding 0 0.5em
    display flex
    align-items baseline
    justify-content space-between
    h3
      margin 0
      font-size 0.9em
    button
      font-size 1.5em
      background transparent
      border 0
      box-shadow none
      color $color
      padding 0
      line-height 1.5
      cursor pointer
  ul
    margin 0
    padding 0 1em
    list-style none
    flex-grow 1
    overflow scroll
  .sig-item
    position relative
    width 100%
    padding 0 0 20px 20px
    margin 1em 0
    overflow hidden
    text-overflow ellipsis
    font-size 0.9em
    i
      font-style normal
      position absolute
      left 0
      cursor pointer
    .sig-item-op
      position absolute
      margin-top 3px
      font-size 0.8em
      a
        display none
        color rgba($color, 0.5)
        cursor pointer
      * + a
        margin-left 1em
    &:hover
      .sig-item-op
        a
          display inline
</style>

<style lang="stylus">
@import "../assets/themes/dark.styl"
body.dark-theme
  .wallet
    .sig-actions
      button
        color $color
    .sig-item
      .sig-item-op
        a
          color rgba($color, 0.5)
</style>
