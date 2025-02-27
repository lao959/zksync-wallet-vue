<template>
  <div class="chooseTokenBlock">
    <div v-if="loading === true" class="centerBlock">
      <loader />
    </div>
    <template v-else>
      <div class="searchContainer">
        <i-input ref="tokenSymbolInput" v-model="search" :placeholder="`Filter balances in ${tokensType}`" maxlength="10">
          <i>
            <v-icon slot="prefix" name="ri-search-line" />
          </i>
        </i-input>
        <div class="updateBtn" :class="{ disabled: spinnerLoading }" @click="getTokenList(true)">
          <i-tooltip placement="left">
            <v-icon v-if="spinnerLoading" name="ri-loader-5-line" class="spin-animation" />
            <v-icon v-else name="ri-restart-line" />
            <template slot="body">Update {{ tokensType }} balances</template>
          </i-tooltip>
        </div>
      </div>
      <div class="tokenListContainer genericListContainer">
        <div v-for="item in displayedList" :key="item.symbol" class="tokenItem" @click="chooseToken(item)">
          <div class="tokenSymbol">{{ item.symbol }}</div>
          <div class="rightSide">
            <div class="balance">{{ item.balance }}</div>
          </div>
        </div>
        <div v-if="search && displayedList.length === 0" class="centerBlock">
          <span>
            Your search <b>"{{ search }}"</b> did not match any tokens
          </span>
        </div>
        <div v-else-if="displayedList.length === 0" class="centerBlock">
          <span>No balances yet. Please make a deposit or request money from someone!</span>
        </div>
      </div>
      <!--suppress JSCheckFunctionSignatures -->
      <i-button block class="_margin-top-1" link size="lg" variant="secondary" @click="$accessor.openModal('NoTokenFound')"> Can't find a token?</i-button>
      <no-token-found />
    </template>
  </div>
</template>

<script lang="ts">
import NoTokenFound from "@/blocks/modals/NoTokenFound.vue";
import utils from "@/plugins/utils";
import { ZkInBalance } from "@/types/lib";

import Vue, { PropOptions } from "vue";

type TokensType = "L1" | "L2";

export default Vue.extend({
  components: {
    NoTokenFound,
  },
  props: {
    onlyAllowed: {
      type: Boolean,
      default: false,
      required: false,
    },
    tokensType: {
      type: String,
      default: "L2",
      required: false,
    } as PropOptions<TokensType>,
  },
  data() {
    return {
      search: "",
      loading: true,
      spinnerLoading: false,
    };
  },
  computed: {
    balances(): ZkInBalance[] {
      return this.tokensType === "L2" ? this.$accessor.wallet.getzkBalances : this.$accessor.wallet.getInitialBalances;
    },
    displayedList(): ZkInBalance[] {
      let list: ZkInBalance[] = utils.searchInArr(this.search, this.balances, (e) => (e as ZkInBalance).symbol) as ZkInBalance[];
      if (this.onlyAllowed) {
        list = list.filter((e) => !e.restricted);
      }
      return list;
    },
  },
  async mounted() {
    await this.getTokenList();
    this.loading = false;
  },
  methods: {
    chooseToken(token: ZkInBalance): void {
      this.$emit("chosen", token);
    },
    async getTokenList(force = false): Promise<void> {
      this.spinnerLoading = true;
      if (this.tokensType === "L2") {
        await this.$accessor.wallet.requestZkBalances({ accountState: undefined, force });
      } else {
        await this.$accessor.wallet.requestInitialBalances(force);
      }
      this.spinnerLoading = false;
    },
  },
});
</script>
