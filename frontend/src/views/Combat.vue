<template>
  <div class="body main-font">
    <character-bar
        v-if="!featureFlagStakeOnly && currentCharacterId !== null"
    />
    <div v-if="ownWeapons.length > 0 && ownCharacters.length > 0">
      <div class="row" v-if="error !== null">
        <div class="col error">Error: {{ error }}</div>
      </div>
<!-- v-if="fightResults[0]" -->
      <b-modal id="fightResultsModal" hide-footer>
         <link rel="prefetch" as="image" href="/img/Congrats.5ac13bd5.gif">
          <img src="../assets/v2/Congrats.gif" class="background-win">
          <div class="icon-close" @click="$bvModal.hide('fightResultsModal')"></div>
          <div class="title-results">{{titleResults}}</div>
          <CombatResults v-if="resultsAvailable" :results="fightResults" />
          <button class="mt-3 btn-buy btn-close-fight-results" block @click="$bvModal.hide('fightResultsModal')">Close</button>
      </b-modal>

      <b-modal id="weaponDurabilityWarning" hide-footer title="Not enough durability">
        <div>Not enough durability, choose others weapon</div>
        <b-button class="mt-3 btn-buy" block @click="$bvModal.hide('weaponDurabilityWarning')">Close</b-button>
      </b-modal>

      <div class="row">
        <div class="col">
          <div class="message-box" v-if="!currentCharacter">You need to select a character to do battle.</div>
          <div class="row">
            <div class="col-12 col-md-2 offset-md-5 text-center">
              <div class="message-box flex-column" v-if="currentCharacter && currentCharacterStamina < staminaPerFight">
                You need {{ staminaPerFight }} stamina to do battle.
                <h4>Stamina Cost Per Fight</h4>
                <b-form-select v-model="fightMultiplier" :options='setStaminaSelectorValues()' @change="setFightMultiplier()" class="ml-3">
                </b-form-select>
              </div>
            </div>
          </div>

          <div class="message-box" v-if="selectedWeaponId && !weaponHasDurability(selectedWeaponId)">This weapon does not have enough durability.</div>

          <div class="message-box" v-if="timeMinutes === 59 && timeSeconds >= 30">You cannot do battle during the last 30 seconds of the hour. Stand fast!</div>
        </div>
      </div>

      <!-- <img src="../assets/divider7.png" class="info-divider enemy-divider" /> -->

      <div class="row" v-if="currentCharacterStamina >= staminaPerFight">
        <div class="col">
          <div class="row">
            <div class="col">
              <div class="waiting" v-if="waitingResults" margin="auto">
                <i class="fas fa-spinner fa-spin"></i>
                Waiting for fight results...
              </div>
            </div>
          </div>
          <div class="combat-enemy-container">
            <div class="col weapon-selection">
              <div class="header-row">
                <div class="info-weapon">
                  <div :class="['col-12', selectedWeaponId ? 'col-md-6 offset-md-3' : 'col-md-2 offset-md-5']">
                    <div class="dropdown-icon"></div>
                    <h4>Stamina Cost per Fight</h4>
                    <b-form-select v-model="fightMultiplier" :options='setStaminaSelectorValues()' @change="setFightMultiplier()">
                    </b-form-select>
                  </div>
                </div>

                <!-- <div class="header-row weapon-header">
                  <b>Choose a weapon</b>
                  <Hint
                    text="Your weapon multiplies your power<br>
                    <br>+Stats determine the multiplier
                    <br>Stat element match with character gives greater bonus"
                  />
                </div> -->

                <div v-if="selectedWeaponId" class="weapon-icon-wrapper">
                  <weapon-icon class="weapon-icon" :weapon="selectedWeapon" />
                </div>

                <b-button v-if="selectedWeaponId" class="ml-3 choose_weapon" @click="selectedWeaponId = null" id="gtag-link-others" tagname="choose_weapon">
                  + Choose New Weapon
                </b-button>
              </div>

              <weapon-grid v-if="!selectedWeaponId" v-model="selectedWeaponId" :checkForDurability="true" />
            </div>
            <div class="nav-line boder"></div>
            <div class="row mb-3 enemy-container" v-if="targets.length > 0">
              <div class="col-12 text-center">
                <div class="combat-hints">
                  <!-- <span class="fire-icon" /> » <span class="earth-icon" /> » <span class="lightning-icon" /> » <span class="water-icon" /> » -->
                  <!-- <span class="fire-icon" /> -->
                  <div class="property-icon"></div>
                  <!-- <Hint
                    text="The elements affect power:<br>
                    <br>Character vs Enemy: bonus or penalty as shown above
                    <br>Character and Weapon match gives bonus"
                  /> -->
                </div>
              </div>

              <div class="row list-enemy">
                <div class="encounter" v-for="(e, i) in targets" :key="i">
                <div class="encounter-container">
                <div class="enemy-character">
                    <div class="encounter-element">
                      <span :class="getCharacterTrait(e.trait).toLowerCase() + '-icon element-icon circle-element' " />
                    </div>

                    <div class="enemy-id">ID {{i}}</div>

                    <div class="">
                      <img class="mx-auto enemy-img" :src="getEnemyArt(e.power)" alt="Enemy" />
                      <img class="enemy-around-img" :src="getEnemyArtAround(e.trait)" alt="Enemy Around">
                    </div>

                    <div class="encounter-power">
                      {{ e.power }} Power
                    </div>

                    <div class="xp-gain">
                      +{{getPotentialXp(e)}} XP
                    </div>
                </div>

                <div class="victory-chance">
                  {{ getWinChance(e.power, e.trait) }} Victory
                </div>
                <big-button
                    style="margin: 0 auto"
                    class="encounter-button btn-styled"
                    :mainText="`FIGHT`"
                    :disabled="(timeMinutes === 59 && timeSeconds >= 30) || waitingResults || !weaponHasDurability(selectedWeaponId) || !charHasStamina()"
                    @click="onClickEncounter(e)"
                  />
                <p v-if="isLoadingTargets">Loading...</p>
                </div>
              </div>
              </div>
            </div>
          </div>
        </div>
      </div>

      <div></div>
    </div>

    <div class="blank-slate" v-if="ownWeapons.length === 0 || ownCharacters.length === 0">
      <div v-if="ownWeapons.length === 0">You do not currently have any weapons. You can buy one at the CryptoWars shop.</div>

      <div v-if="ownCharacters.length === 0">You do not currently have any characters. You can recruit one at the Plaza.</div>
    </div>
  </div>
</template>

<script>
// import Character from "../components/Character.vue";
import BigButton from '../components/BigButton.vue';
import WeaponGrid from '../components/smart/WeaponGrid.vue';
import {getEnemyArtAround} from '../enemy-art-around';
import { getEnemyArt } from '../enemy-art';
import { CharacterPower, CharacterTrait, GetTotalMultiplierForTrait, WeaponElement } from '../interfaces';
// import Hint from '../components/Hint.vue';
import CombatResults from '../components/CombatResults.vue';
import { toBN, fromWeiEther } from '../utils/common';
import WeaponIcon from '../components/WeaponIcon.vue';
import { mapActions, mapGetters, mapState, mapMutations } from 'vuex';
import CharacterBar from "../components/CharacterBar.vue";

export default {
  data() {
    return {
      selectedWeaponId: null,
      error: null,
      waitingResults: false,
      resultsAvailable: false,
      fightResults: null,
      intervalSeconds: null,
      intervalMinutes: null,
      timeSeconds: null,
      timeMinutes: null,
      fightXpGain: 16,
      selectedWeapon: null,
      fightMultiplier: Number(localStorage.getItem('fightMultiplier')),
      staminaPerFight: 40,
      titleResults: "",
    };
  },

  created() {
    this.intervalSeconds = setInterval(() => (this.timeSeconds = new Date().getSeconds()), 5000);
    this.intervalMinutes = setInterval(() => (this.timeMinutes = new Date().getMinutes()), 20000);
    this.staminaPerFight = 40 * Number(localStorage.getItem('fightMultiplier'));
  },

  computed: {
    ...mapState(['currentCharacterId']),
    ...mapGetters([
      'getTargetsByCharacterIdAndWeaponId',
      'ownCharacters',
      'ownWeapons',
      'currentCharacter',
      'currentCharacterStamina',
      'getWeaponDurability',
      'fightGasOffset',
      'fightBaseline',
    ]),

    targets() {
      return this.getTargetsByCharacterIdAndWeaponId(this.currentCharacterId, this.selectedWeaponId);
    },

    isLoadingTargets() {
      return this.targets.length === 0 && this.currentCharacterId && this.selectedWeaponId;
    },

    selections() {
      return [this.currentCharacterId, this.selectedWeaponId];
    },

    updateResults() {
      return [this.fightResults, this.error];
    },
  },

  watch: {
    async selections([characterId, weaponId]) {
      if (!this.ownWeapons.filter(Boolean).find((weapon) => weapon.id === weaponId)) {
        this.selectedWeaponId = null;
      }
      await this.fetchTargets({ characterId, weaponId });
    },

    async updateResults([fightResults, error]) {
      this.resultsAvailable = fightResults !== null;
      this.waitingResults = fightResults === null && error === null;
      this.setIsInCombat(this.waitingResults);
      if (this.resultsAvailable && error === null) this.$bvModal.show('fightResultsModal');
      if(this.fightResults[0] === true){
        this.titleResults = "Congratulation!";
      }
      else{
        this.titleResults = "Better luck Next Time";
      }
    },
  },


  methods: {
    ...mapActions(['fetchTargets', 'doEncounter', 'fetchFightRewardSkill', 'fetchFightRewardXp', 'getXPRewardsIfWin']),
    ...mapMutations(['setIsInCombat']),
    getEnemyArt,
    getEnemyArtAround,
    weaponHasDurability(id) {
      return this.getWeaponDurability(id) >= this.fightMultiplier * 3;
    },
    charHasStamina(){
      return this.currentCharacterStamina >= this.staminaPerFight;
    },
    getCharacterTrait(trait) {
      return CharacterTrait[trait];
    },
    getWinChance(enemyPower, enemyElement) {
      const characterPower = CharacterPower(this.currentCharacter.level);
      const playerElement = parseInt(this.currentCharacter.trait, 10);
      const selectedWeapon = this.ownWeapons.filter(Boolean).find((weapon) => weapon.id === this.selectedWeaponId);
      this.selectedWeapon = selectedWeapon;
      const weaponElement = parseInt(WeaponElement[selectedWeapon.element], 10);
      const weaponMultiplier = GetTotalMultiplierForTrait(selectedWeapon, playerElement);
      const totalPower = characterPower * weaponMultiplier + selectedWeapon.bonusPower;
      const totalMultiplier = 1 + 0.075 * (weaponElement === playerElement ? 1 : 0) + 0.075 * this.getElementAdvantage(playerElement, enemyElement);
      const playerMin = totalPower * totalMultiplier * 0.9;
      const playerMax = totalPower * totalMultiplier * 1.1;
      const playerRange = playerMax - playerMin;
      const enemyMin = enemyPower * 0.9;
      const enemyMax = enemyPower * 1.1;
      const enemyRange = enemyMax - enemyMin;
      let rollingTotal = 0;
      // shortcut: if it is impossible for one side to win, just say so
      if (playerMin > enemyMax) return 'Very Likely';
      if (playerMax < enemyMin) return 'Unlikely';

      // case 1: player power is higher than enemy power
      if (playerMin >= enemyMin) {
        // case 1: enemy roll is lower than player's minimum
        rollingTotal = (playerMin - enemyMin) / enemyRange;
        // case 2: 1 is not true, and player roll is higher than enemy maximum
        rollingTotal += (1 - rollingTotal) * ((playerMax - enemyMax) / playerRange);
        // case 3: 1 and 2 are not true, both values are in the overlap range. Since values are basically continuous, we assume 50%
        rollingTotal += (1 - rollingTotal) * 0.5;
      } // otherwise, enemy power is higher
      else {
        // case 1: player rolls below enemy minimum
        rollingTotal = (enemyMin - playerMin) / playerRange;
        // case 2: enemy rolls above player maximum
        rollingTotal += (1 - rollingTotal) * ((enemyMax - playerMax) / enemyRange);
        // case 3: 1 and 2 are not true, both values are in the overlap range
        rollingTotal += (1 - rollingTotal) * 0.5;
        //since this is chance the enemy wins, we negate it
        rollingTotal = 1 - rollingTotal;
      }
      if (rollingTotal <= 0.3) return 'Unlikely';
      if (rollingTotal <= 0.5) return 'Possible';
      if (rollingTotal <= 0.7) return 'Likely';
      return 'Very Likely';
    },
    getElementAdvantage(playerElement, enemyElement) {
      if ((playerElement + 1) % 4 === enemyElement) return 1;
      if ((enemyElement + 1) % 4 === playerElement) return -1;
      return 0;
    },
    async onClickEncounter(targetToFight) {
      if (this.selectedWeaponId === null || this.currentCharacterId === null) {
        return;
      }

      this.waitingResults = true;

      // Force a quick refresh of targets
      await this.fetchTargets({ characterId: this.currentCharacterId, weaponId: this.selectedWeaponId });
      // If the targets list no longer contains the chosen target, return so a new target can be chosen
      if (!this.targets.find((target) => target.original === targetToFight.original)) {
        this.waitingResults = false;
        return;
      }

      this.fightResults = null;
      this.error = null;
      this.setIsInCombat(this.waitingResults);

      try {
        const results = await this.doEncounter({
          characterId: this.currentCharacterId,
          weaponId: this.selectedWeaponId,
          targetString: targetToFight.original,
          fightMultiplier: this.fightMultiplier,
        });

        this.fightResults = results;

        await this.fetchFightRewardSkill();
        await this.fetchFightRewardXp();

        this.error = null;
      } catch (e) {
        console.error(e);
        this.error = e.message;
      }
    },

    formattedSkill(skill) {
      const skillBalance = fromWeiEther(skill, 'ether');
      return `${toBN(skillBalance).toFixed(6)} xBlade`;
    },

    getPotentialXp(targetToFight) {
      const characterPower = CharacterPower(this.currentCharacter.level);
      const playerElement = parseInt(this.currentCharacter.trait, 10);
      const selectedWeapon = this.ownWeapons.filter(Boolean).find((weapon) => weapon.id === this.selectedWeaponId);
      const weaponMultiplier = GetTotalMultiplierForTrait(selectedWeapon, playerElement);
      const totalPower = characterPower * weaponMultiplier + selectedWeapon.bonusPower;

      //Formula taken from getXpGainForFight funtion of CryptoWars.sol
      return Math.floor((targetToFight.power / totalPower) * this.fightXpGain) * this.fightMultiplier;
    },

    setFightMultiplier() {
      localStorage.setItem('fightMultiplier', this.fightMultiplier.toString());
    },

    setStaminaSelectorValues() {
      if(this.currentCharacterStamina < 40) {
        return [{ value: this.fightMultiplier, text: 'You need more stamina to fight!', disabled: true}];
      }

      const choices = [
        {value: null, text: 'Please select Stamina Cost per Fight', disabled: true},
      ];

      const addChoices = [];

      if(this.currentCharacterStamina >= 200) {
        addChoices.push({ value: 5, text: 200 });
      }

      if(this.currentCharacterStamina >= 160) {
        addChoices.push({ value: 4, text: 160 });
      }

      if(this.currentCharacterStamina >= 120) {
        addChoices.push({ value: 3, text: 120 });
      }

      if(this.currentCharacterStamina >= 80) {
        addChoices.push({ value: 2, text: 80 });
      }

      if(this.currentCharacterStamina >= 40) {
        addChoices.push({ value: 1, text: 40 });
      }

      choices.push(...addChoices.reverse());

      return choices;
    },
  },

  components: {
    BigButton,
    WeaponGrid,
    // Hint,
    CombatResults,
    WeaponIcon,
    CharacterBar,
  },
};
</script>

<style scoped>
.background-win{
  background-size: 100%;
  background-repeat: no-repeat;
  width: 473px;
  height: 300px;
  position: fixed;
}

#fightResultsModal .icon-close{
  background-image: url(../assets/v2/icon-close.svg);
  width: 50px;
  height: 47px;
  position: relative;
  right: -24em;
  z-index: 2;
}


.title-results{
  font-size: 1.3em;
  color: #fff;
  text-align: center;
  margin-top: 30px;
}

.btn-close-fight-results{
  border-radius: 0;
  border: none;
  background-image: url(../assets/v2/btn-fight.png);
  width: 130px;
  height: 50px;
  background-size: cover;
  margin: 0 auto;
  background-color: transparent;
  margin: 0 auto;
  display: block;
}

.enemy-character {
  position: relative;
  width: 17em;
  height: 23em;
  background-position: left;
  background-repeat: no-repeat;
  background-size: contain;
  background-image: url('../assets/images/bg-item-top.png');
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
}

.enemy-around{
  background-repeat: no-repeat;
  background-size: contain;
  background-image: url('../assets/images/bg-item-top.png');
}

.encounter img {
  width: 140px;
}

.weapon-header > b {
  font-size: 1.8em;
}

.payout-info {
  margin: auto;
  text-align: center;
  padding-top: 1em;
  font-size: 1.5em;

  display: flex;
  justify-content: center;
  align-items: center;
}

.combat-hints {
  margin: auto;
  text-align: center;
  padding-right: 1em;
  padding-left: 1em;
  font-size: 2em;
  display: flex;
  justify-content: center;
  align-items: center;
}

.combat-hints .property-icon{
  background-repeat: no-repeat;
  background-size: contain;
  background-image: url(../assets/v2/Property-system.svg);
  width: 360px;
  height: 80px;
}

.combat-hints .hint {
  margin-left: 10px;
}

.waiting {
  font-size: 2em;
  margin: auto;
  text-align: center;
}

.header-row {
  display: flex;
  align-items: center;
}

.header-row h1 {
  margin-left: 10px;
  margin-bottom: 5px;
}

.header-row .hint {
  font-size: 2em;
}

.info-weapon > div > div{
  display: flex;
  justify-content: center;
  position: relative;
}

.info-weapon > div > div::before{
  content: "";
  background-repeat: no-repeat;
  background-size: contain;
  background-image: url('../assets/v2/dropdown-icon.svg');
  width: 20px;
  height: 20px;
  position: relative;
  top: 123px;
  right: -120px;
  z-index: 1;
}

.message-box {
  display: flex;
  justify-content: center;
  width: 100%;
  font-size: 2em;
}

.combat-enemy-container h4[data-v-067077ae] {
    font-size: 1.2rem;
    font-weight: 300;
    margin: 20px 0;
}

.custom-select{
  background-color: #000;
  border: 1px solid #5bc7f5;
  width: 272px;
  height: 60px;
  border-radius: 10px;
  margin-top: 15px;
}

div.encounter.text-center {
  flex-basis: auto !important;
}

.weapon-icon-wrapper {
  /* background: rgba(255, 255, 255, 0.1); */
  width: 20em;
  height: 24em;
  margin-top: 55px !important;
}

/* .glow-0[data-v-5471bc65]{
  box-shadow: none!important;
} */

.list-enemy{
  display: flex;
  justify-content: center;
  width: 100%;
  /* padding: 0px 14px; */
  margin-top: 20px;
}

.encounter-container {
  position: relative;
}

.encounter {
  display: flex;
  justify-content: center;
  margin-bottom: 62px;
}

.weapon[data-v-34f9e3b8] {
  overflow: auto;
}

/* .xp-gain,
.encounter-power {
  color: #F58B5B !important;
} */

.xp-gain,
.encounter-power,
.encounter-element,
.victory-chance {
  position: absolute;
}

.encounter-element {
  top: 12px;
  left: 12px;
  font-size: 20px;
}

.encounter-power {
  bottom: 50px;
  font-size: 1.2em;
}

.xp-gain {
  bottom: 25px;
  font-size: 1em;
}

.victory-chance {
  left: 0;
  right: 0;
  text-align: center;
  font-size: 1.1em;
  font-weight: 600;
  text-shadow: -1px 0 #000, 0 1px #000, 1px 0 #000, 0 -1px #000;
}

/* Mobile Support Classes*/
.mobile-divider-wrapper {
  width: 100%;
  display: flex;
}

.mobile-divider {
  margin: auto;
}

.combat-enemy-container {
  display: flex;
  margin-bottom: 50px;
  margin-top: 50px;
}

.combat-enemy-container .weapon-icon-wrapper{
  height: 24rem;
  width: 19.5rem;
}

.combat-enemy-container .small-durability-bar{
  top: 36px;
}

.combat-enemy-container h4{
  font-size: 1.3rem;
}

.nav-line.boder{
  width: 2px;
  height: 668px;
  /* position: relative;
  left: -28px;
  top: -120px; */
  background-repeat: no-repeat;
  background-size: cover;
  background-image: url('../assets/v2/boder.png');
}

.enemy-container {
  flex: 3;
  /* border-left: 1px solid #F58B5B; */
}

.enemy-divider {
  margin-top: 30px;
}

.enemy-list {
  display: flex;
  flex-wrap: wrap;
  padding-left: 30px;
  padding-right: 30px;
}

.weapon-selection {
  /* border-right: 1px solid #F58B5B; */
}

.weapon-header {
  justify-content: center;
  margin-bottom: 20px;
  margin-top: 20px;
}

.enemy-energy {
  top: -30px;
  position: relative;
}

.enemy-id{
  position: absolute;
  top: 22px;
  right: 38px;
  font-size: 1.3em;
}

h1 {
  font-weight: 900 !important;
  text-align: center;
  font-size: 3vw;
  padding-top: 0px;
}

button.encounter-button {
  display: flex;
  position: relative;
  /* margin-right: 0; */
  top: 3.3vw;
  text-align: center;
  line-height: 1;
  justify-content: center;
  align-items: center;
  background-image: url(../assets/v2/btn-fight.png);
  background-repeat: no-repeat;
  background-size: contain;
  border: none;
  background-color: transparent;
  height: 60px;
  width: 152px !important;
  overflow: hidden;
  margin: 0 auto;
}

button.encounter-button:hover{
  background-image: url(../assets/v2/Btn-fight-opcity.png);
  transition: 0.9s;
  background-color: transparent;
}

.encounter-button:active{
  margin-left: 0px;
}

button.btn.button.main-font.dark-bg-text.encounter-button.btn-styled.encounter-button > h1{
  margin: 0;
}
.enemy-img {
  position: relative;
  top: 70px;
  z-index: 1;
}

.enemy-around-img{
  position: relative;
  top: -129px;
}

.choose_weapon{
  width: 15rem;
  height: 46px;
  background-color: transparent !important;
  border: none !important;
  background-image: url(../assets/v2/Choose-new-weapon.svg);
  background-repeat: no-repeat;
  background-size: contain;
  border-radius: 0;
  margin: 0;
  font-size: 1.2em;
  padding: 0;
  margin: 0 auto;
}

@media (max-width: 1334px) {
  .enemy-list {
    flex-flow: row wrap;
    align-items: center;
  }
  .enemy-list > .enemy-list-child{
     flex-basis: 50%;
  }
  .encounter-button {
    margin-top: 1.35em;
  }
  .nav-line.boder{
    height: 0;
  }
}

/* Needed to asjust image size, not just image column-size and other classes to accommodate that */
@media all and (max-width: 767.98px) {
  /* .encounter img {
    width: calc(100% - 60px);
  } */
  .enemy-list{
    flex-direction:column;
    align-items:center;
  }
  .combat-enemy-container {
    flex-direction: column;
    align-items: center;
  }
  .weapon-selection {
    border-right: none;
  }
  .results-panel {
    width: 100%;
  }
  .nav-line.boder{
    height: 0;
  }
}
.hint.has-tooltip {
  font-size: 1.8rem;
  display: inline-block;
  margin-left: 10px;
}
.dark-bg-text {
  width: 100% !important;
}
.content {
  padding: 0 !important;
}

/* - */
.combat-hints {
  margin-top: 30px;
}
.ml-3 {
  margin-left: 0px !important;
}
.header-row {
  display: block;
  text-align: center;
}
.weapon-icon-wrapper {
  margin: 0 auto;
  width: 14em;
  height: 26em;
}

.modal-content {
  border-radius: 20px;
  width: 500px;
}

@media (max-width: 767.98px){
  .button.encounter-button{
    top: 10vw;
  }
  .small-durability-bar{
    top: 35px
  }
  .nav-line.boder{
    height: 0;
  }
  .content{
    padding: 0;
  }
}

@media (max-width: 575.98px) {
  .show-reforged {
    width: 100%;
    justify-content: center;
    display: block;
  }
  .row{
    margin: 0;
  }
  .nav-line.boder{
    height: 0;
  }
}
.element-icon {
  width: 3em;
  height: 3em;
}
@media (min-width: 768px){
  .offset-md-3 {
    margin: 0;
  }
  .col-md-6 {
    max-width: 100%;
  }
  .col-md-2 {
    max-width: 100%;
  }
  .offset-md-5 {
    margin-left: 0;
  }
}

.circle-element {
  width: 3.3rem;
  height: 3.3rem;
  border-radius: 50%;
  padding: 0.5rem;
}
</style>
