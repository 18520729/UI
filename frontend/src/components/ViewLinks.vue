<template>
  <b-navbar-nav>
    <router-link :to="{ name: 'lobby' }" exact class="nav-link mr-4"
    v-if="!featureFlagStakeOnly">
      <li class="nav-item nav-top-links lobby">
        <span class="gtag-link-others" tagname="plaza_screen">Lobby</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <router-link :to="{ name: 'arena' }" exact class="nav-link mr-4 ml-4"
    v-if="!featureFlagStakeOnly">
      <li class="nav-item nav-top-links nav-icon-box">
        <img src="../assets/v2/arena_icon.svg" alt="" class="nav-icon">
        <span class="gtag-link-others" tagname="plaza_screen">Arena</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <div class="nav-link  mr-4 ml-4">
      <li class="nav-item nav-top-links ">
        <a href="https://www.cryptowar.network/buy" target="_blank" class="gtag-link-others nav-icon-box" tagname="shop_screen">
          <img src="../assets/v2/ido_icon.svg" alt="" class="nav-icon">
          <span class="gtag-link-others" tagname="plaza_screen">Buy IDO</span>
        </a>
      </li>
    </div>

    <li class="nav-line mr-4"></li>

    <router-link :to="{ name: 'referral' }" exact class="nav-link  mr-3 ml-4">
      <li class="nav-item nav-top-links nav-img-item-box">
        <span class="nav-img-item">
          <img src="../assets/v2/refer_icon.svg" alt="Referral" class="nav-img">
        </span>
        <span class="nav-item-hover">Referral</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <!-- <router-link :to="{ name: 'plaza' }" exact class="nav-link"
    v-if="!featureFlagStakeOnly">
      <li class="nav-item nav-top-links">
        <span class="gtag-link-others" tagname="plaza_screen">Plaza</span>
      </li>
    </router-link> -->

    <router-link :to="{ name: 'blacksmith' }" exact class="nav-link mr-3 ml-3"   v-if="!featureFlagStakeOnly">
      <li class="nav-item nav-top-links nav-img-item-box" >
        <span class="nav-img-item">
          <img src="../assets/v2/inventory.svg" alt="Inventory" class="nav-img">
        </span>
         <span class="nav-item-hover">Inventory</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <router-link :to="{ name: 'market' }" exact class="nav-link mr-3 ml-3" v-if="!featureFlagStakeOnly && featureFlagMarket">
      <li class="nav-item nav-top-links nav-img-item-box">
        <span class="nav-img-item">
          <img src="../assets/v2/marketplace_icon.svg" alt="Market Place" class="nav-img">
        </span>
        <span class="nav-item-hover">Market Place</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <router-link :to="{ name: 'shop' }" exact class="nav-link mr-3 ml-3">
      <li class="nav-item nav-top-links nav-img-item-box">
        <span class="nav-img-item">
          <img src="../assets/v2/shop_icon.svg" alt="Shop" class="nav-img">
        </span>
        <span class="nav-item-hover">Shop</span>
      </li>
    </router-link>

    <li class="nav-line"></li>

    <router-link :to="{ name: 'blacksmith' }" exact class="nav-link mr-3 ml-3">
      <li class="nav-item nav-top-links nav-img-item-box">
        <span class="nav-img-item">
          <img src="../assets/v2/blacksmith_icon.svg" alt="Black Smith" class="nav-img">
        </span>
        <span class="nav-item-hover">Black Smith</span>
      </li>
    </router-link>

    <!-- <router-link :to="{ name: 'combat' }" exact class="nav-link"  v-if="!featureFlagStakeOnly">
      <li class="nav-item nav-top-links" >
        <span class="gtag-link-others" tagname="combat_screen">Combat</span>
      </li>
    </router-link> -->

    <!-- <router-link :to="{ name: 'raid' }" exact class="nav-link"  v-if="!featureFlagStakeOnly && featureFlagRaid">
      <li class="nav-item nav-top-links">
        <span class="gtag-link-others" tagname="raid_screen">Raid</span>
      </li>
    </router-link> -->

    <!-- <router-link :to="{ name: 'select-stake-type' }" exact class="nav-link">
      <li class="nav-item nav-top-links">
        <span class="gtag-link-others" tagname="stake_screen">Stake</span>
      </li>
    </router-link> -->


    <!-- <router-link :to="{ name: 'P2P' }" exact class="nav-link">
      <li class="nav-item nav-top-links">
        <span class="gtag-link-others" tagname="pvp_screen">PvP</span>
      </li>
    </router-link> -->

    <!-- <div exact class="nav-link">
      <li class="nav-item nav-top-links">
        <a href="https://www.cryptowar.network/buy" target="_blank" class="gtag-link-others" tagname="shop_screen">Buy IDO</a>
      </li>
    </div>-->

  </b-navbar-nav>
</template>

<script>
import { market as featureFlagMarket, portal as featureFlagPortal } from '../feature-flags';

export default {
  inject: ['featureFlagStakeOnly', 'featureFlagRaid'],

  computed: {
    featureFlagMarket() {
      return featureFlagMarket;
    },
    featureFlagPortal() {
      return featureFlagPortal;
    }
  },
};
</script>

<style scoped>
a {
  font-weight: bold;
  color: #ffffff;
}
a:hover {
  color: #F58B5B;
}

.nav-top-links > span {
  color : #ffffff;
  font-size: 1.1em;
  padding: 0px 5px 0px 5px;
}
.lobby{
  background: url('../assets/v2/lobby.svg') no-repeat center;
  background-size: contain;
  width: 210px;
  height: 65px;
  display: flex;
  justify-content: center;
  align-items: center;
  text-transform: uppercase;
}

.lobby:hover{
  background: url('../assets/v2/lobby_hover.svg') no-repeat center;
  background-size: contain;
}

a.router-link-active .lobby .gtag-link-others{
  color: #fff;
  margin-right: 0px;
  margin-top: -4px;
}
.nav-icon-box{
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-icon-box .nav-icon{
  height: 1.1em;
}
.nav-icon-box .gtag-link-others{
  padding-left: 5px;
  margin-right: 0;
}

.nav-img-item-box{
  position: relative;
  display: flex;
  align-items: center;
  justify-content: center;
}

.nav-img-item{
  display: flex;
  width: 47px;
  height: 47px;
  background-image: linear-gradient(to bottom, #F56F56, #F56F56, #F26B65, #EB618E, #E151D0, #DD4BEC);
  align-items: center;
  border-radius: 14px;
  justify-content: center;
}
.nav-img-item img{
  max-width: 100%;
}

.nav-img-item-box .nav-item-hover{
  position: absolute;
  display: none;
  width: 135px;
  height: 50px;
  background: url("../assets/v2/nav-hover.svg") no-repeat 50% 50%;
  background-size: cover;
  top: 100%;
  justify-content: center;
  align-items: center;
  font-size: 0.8em;
  z-index: 100;
}
.nav-img-item-box:hover .nav-item-hover{
  display: flex;
}
.nav-line{
  width: 2px;
  height: 60px;
  background-image: linear-gradient(to bottom, #000000ff, #A9A9A9, #000000ff);
}

</style>
