<template>
  <div class="body main-font">
    <b-navbar>
      <!-- <b-icon-exclamation-circle-fill class="rewards-claimable-icon" scale="1.2"
      variant="success" :hidden="!canClaimTokens && !canClaimXp" v-tooltip.bottom="'Rewards ready to claim!'"/> -->

      <b-nav-item
        :disabled="!canClaimTokens"
        @click="claimSkill(ClaimStage.Claim)"
        ><!-- moved gtag-link below b-nav-item -->
        <span
          class="gtag-link-others claim-reward-text"
          tagname="claim_skill"
          v-tooltip.bottom="
            'Tax is being reduced by 1% per day.' + getTaxTimerNextTick
          "
        >
          Claim rewards {{ formattedXBladeReward }} xBlade Early Withdraw Tax
          {{ formattedRewardsClaimTax }}
          <!-- <b-icon-question-circle class="centered-icon" scale="0.8"/> -->
        </span>
      </b-nav-item>

      <b-nav-item :disabled="!canClaimXp" @click="onClaimXp">
        <b-button
          class="gtag-link-others"
          v-html="`Claim Heroes XP`"
          v-tooltip.bottom="formattedXpRewards"
        ></b-button>
      </b-nav-item>
    </b-navbar>

    <b-modal
      class="centered-modal"
      ref="stake-suggestion-modal"
      title="Stake Skill"
      @ok="$router.push({ name: 'select-stake-type' })"
      ok-only
      ok-title="Go to Stake"
    >
      You can avoid paying the 15% tax by staking unclaimed skill rewards for 7
      days. If you stake your xBlade now, we'll give you a 50% bonus in-game
      only xBlade that you can use right away!
      <a href="#" @click="claimSkill(ClaimStage.Claim)">
        <br />No thanks, I'd rather
        {{
          this.rewardsClaimTaxAsFactorBN > 0
            ? "pay " + this.formattedTaxAmount + " in taxes and "
            : ""
        }}forfeit my bonus
      </a>
    </b-modal>
    <b-modal
      class="centered-modal"
      ref="claim-confirmation-modal"
      title="Claim xBlade Rewards"
      size="lg"
      :ok-title="`OK, I'll claim and miss out ${this.formattedTaxAmount} xBlade tax`"
      @ok="onClaimTokens()"
    >
      You are about to
      {{
        this.rewardsClaimTaxAsFactorBN > 0
          ? "pay " +
            formattedRewardsClaimTax +
            " tax for early withdrawal, costing you " +
            this.formattedTaxAmount
          : ""
      }}
      . Are you sure you wish to continue? <b>This action cannot be undone.</b>
      <div>
        <hr class="hr-divider" />
        Hold Reminder:<br />
        A percentage of your earning goes back to the community,<br />
        <u>if you withdraw early</u>
        <div class="row col-12">
          <div>
            Your early withdraw tax
            <span class="text-danger font-weight-bold"
              >{{ formattedRewardsClaimTax }}
            </span>
          </div>
          <div class="text-left" style="margin-left: 4px">
            . Reduces 1% per day. Reset to 15% after withdraw
          </div>
        </div>
      </div>
    </b-modal>
  </div>
</template>

<script lang="ts">
import Vue from "vue";
import { Accessors } from "vue/types/options";
import { mapActions, mapGetters, mapState } from "vuex";
import BigNumber from "bignumber.js";
import { RequiredXp } from "../../interfaces";
import { ICharacter } from "@/interfaces";
import { toBN, fromWeiEther } from "../../utils/common";
import { secondsToDDHHMMSS } from "../../utils/date-time";
import { getCleanName } from "../../rename-censor";

interface StoreMappedState {
  skillRewards: string;
  xpRewards: Record<string, string>;
  ownedCharacterIds: string[];
  directStakeBonusPercent: number;
}

interface StoreMappedGetters {
  ownCharacters: ICharacter[];
  currentCharacter: ICharacter | null;
  maxRewardsClaimTaxAsFactorBN: BigNumber;
  rewardsClaimTaxAsFactorBN: BigNumber;
  getCharacterName(id: number): string;
}

enum ClaimStage {
  WaxBridge = 0,
  Stake = 1,
  Claim = 2,
}

interface StoreMappedActions {
  claimTokenRewards(): Promise<void>;
  claimXpRewards(): Promise<void>;
}

export default Vue.extend({
  data() {
    return {
      ClaimStage,
    };
  },

  computed: {
    ...(mapState([
      "skillRewards",
      "xpRewards",
      "ownedCharacterIds",
      "directStakeBonusPercent",
    ]) as Accessors<StoreMappedState>),
    ...(mapGetters([
      "ownCharacters",
      "currentCharacter",
      "maxRewardsClaimTaxAsFactorBN",
      "rewardsClaimTaxAsFactorBN",
      "getCharacterName",
    ]) as Accessors<StoreMappedGetters>),

    formattedXBladeReward(): string {
      const skillRewards = fromWeiEther(this.skillRewards);
      return `${toBN(skillRewards).toFixed(2)}`;
    },

    formattedTaxAmount(): string {
      const skillRewards = fromWeiEther(
        (
          parseFloat(this.skillRewards) *
          parseFloat(String(this.rewardsClaimTaxAsFactorBN))
        ).toString()
      );
      return `${toBN(skillRewards).toFixed(4)}`;
    },

    formattedBonusLost(): string {
      const xBladeLost = fromWeiEther(
        (
          (parseFloat(this.skillRewards) * this.directStakeBonusPercent) /
          100
        ).toString()
      );
      return `${toBN(xBladeLost).toFixed(2)}`;
    },

    formattedRewardsClaimTax(): string {
      const frac =
        this.skillRewards === "0"
          ? this.maxRewardsClaimTaxAsFactorBN
          : this.rewardsClaimTaxAsFactorBN;

      return `${frac
        .multipliedBy(100)
        .decimalPlaces(0, BigNumber.ROUND_HALF_UP)}%`;
    },

    getTaxTimerNextTick(): string {
      let frac: BigNumber;

      // if has no skill rewards do not display timer next tick.
      if (this.skillRewards === "0") {
        return "";
      } else {
        frac = this.rewardsClaimTaxAsFactorBN;
      }

      // get 2 decimal values
      const decVal = toBN(
        frac.multipliedBy(100).decimalPlaces(2).toString().split(".")[1]
      );
      // convert to seconds
      const toSec = decVal
        .dividedBy(100)
        .multipliedBy(24)
        .multipliedBy(60)
        .multipliedBy(60);
      // return message
      return ` Next -1% reduction happens in ${secondsToDDHHMMSS(
        toSec.toNumber()
      )}.`;
    },

    xpRewardsForOwnedCharacters(): string[] {
      return this.ownedCharacterIds.map(
        (charaId) => this.xpRewards[charaId] || "0"
      );
    },

    formattedXpRewards(): string {
      return this.xpRewardsForOwnedCharacters
        .map((xp, i) => {
          const currentCharacter = this.currentCharacter || { id: null };
          if (!this.ownCharacters[i]) return `${xp}`;
          return (
            `${
              ((this.ownCharacters[i].xp +
                this.xpRewards[this.ownCharacters[i].id]) as any) >
              RequiredXp(this.ownCharacters[i].level)
                ? ""
                : ""
            }` +
            `${this.getCleanCharacterName(this.ownCharacters[i].id)} ${xp}` +
            `${
              ((this.ownCharacters[i].xp +
                this.xpRewards[this.ownCharacters[i].id]) as any) >
              RequiredXp(this.ownCharacters[i].level)
                ? ""
                : ""
            }` +
            `${this.ownCharacters[i].id === currentCharacter.id ? "" : ""}`
          );
        })
        .join(", ");
    },

    canClaimTokens(): boolean {
      if (toBN(this.skillRewards).lte(0)) {
        return false;
      }

      return true;
    },

    canClaimXp(): boolean {
      const allXpsAreZeroOrLess = this.xpRewardsForOwnedCharacters.every((xp) =>
        toBN(xp).lte(0)
      );
      if (allXpsAreZeroOrLess) {
        return false;
      }

      return true;
    },
  },

  methods: {
    ...(mapActions([
      "addMoreSkill",
      "claimTokenRewards",
      "claimXpRewards",
    ]) as StoreMappedActions),

    async onClaimTokens() {
      if (this.canClaimTokens) {
        await this.claimTokenRewards();
      }
    },

    async onClaimXp() {
      if (this.canClaimXp) {
        await this.claimXpRewards();
      }
    },

    async claimSkill(stage: ClaimStage) {
      // if(stage === ClaimStage.Stake) {
      //   (this.$refs['stake-suggestion-modal'] as any).show();
      // }
      if (stage === ClaimStage.Claim) {
        (this.$refs["stake-suggestion-modal"] as any).hide();
        (this.$refs["claim-confirmation-modal"] as any).show();
      }
    },

    getCleanCharacterName(id: number): string {
      return getCleanName(this.getCharacterName(id));
    },
  },
});
</script>

<style scoped>
.nav-item {
  /* margin-top: -24px; */
}

.nav-item a {
  padding: 0;
  font-size: 1rem;
}

.rewards-claimable-icon {
  margin-right: 5px;
  align-self: center;
}
.tax-timer {
  color: #fff;
}

.claim-reward-modal {
  max-width: 800px;
  background: khaki;
}
.claim-reward-text:hover {
  color: #F58B5B;
}
</style>
