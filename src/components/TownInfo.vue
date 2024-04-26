<template>
  <ul class="信息">
    <li
        class="版本"
        :class="['版本-' + edition.id]"
        :style="{
        backgroundImage: `url(${
          edition.logo && grimoire.isImageOptIn
            ? edition.logo
            : require('../assets/editions/' + edition.id + '.png')
        })`
      }"
    ></li>
    <li v-if="players.length - teams.traveler < 5">
      请添加更多玩家！
    </li>
    <li>
      <span class="元数据" v-if="!edition.isOfficial">
        {{ edition.name }}
        {{ edition.author ? "作者：" + edition.author : "" }}
      </span>
      <span>
        {{ players.length }} <font-awesome-icon class="玩家" icon="users" />
      </span>
      <span>
        {{ teams.alive }}
        <font-awesome-icon class="存活" icon="heartbeat" />
      </span>
      <span>
        {{ teams.votes }} <font-awesome-icon class="投票" icon="vote-yea" />
      </span>
    </li>
    <li v-if="players.length - teams.traveler >= 5">
      <span>
        {{ teams.townsfolk }}
        <font-awesome-icon class="普通村民" icon="user-friends" />
      </span>
      <span>
        {{ teams.outsider }}
        <font-awesome-icon
            class="外来者"
            :icon="teams.outsider > 1 ? 'user-friends' : 'user'"
        />
      </span>
      <span>
        {{ teams.minion }}
        <font-awesome-icon
            class="爪牙"
            :icon="teams.minion > 1 ? 'user-friends' : 'user'"
        />
      </span>
      <span>
        {{ teams.demon }}
        <font-awesome-icon
            class="恶魔"
            :icon="teams.demon > 1 ? 'user-friends' : 'user'"
        />
      </span>
      <span v-if="teams.traveler">
        {{ teams.traveler }}
        <font-awesome-icon
            class="旅行者"
            :icon="teams.traveler > 1 ? 'user-friends' : 'user'"
        />
      </span>
      <span v-if="grimoire.isNight">
                <font-awesome-icon :icon="['fas', 'sun']"/>
      <em v-if="session.sessionId">房间号：{{ session.sessionId }}</em>
      </span>
      <span v-if="!grimoire.isNight">
                <font-awesome-icon :icon="['fas', 'sun']"/>
      <em v-if="session.sessionId">房间号：{{ session.sessionId }}</em>
      </span>
    </li>
  </ul>
</template>

<script>
import gameJSON from "./../game";
import { mapState } from "vuex";

export default {
  computed: {
    teams: function() {
      const { players } = this.$store.state.players;
      const nonTravelers = this.$store.getters["players/nonTravelers"];
      const alive = players.filter(player => player.isDead !== true).length;
      return {
        ...gameJSON[nonTravelers - 5],
        traveler: players.length - nonTravelers,
        alive,
        votes:
            alive +
            players.filter(
                player => player.isDead === true && player.isVoteless !== true
            ).length
      };
    },
    ...mapState(["edition", "grimoire","session"]),
    ...mapState("players", ["players"])
  }
};
</script>

<style lang="scss" scoped>
@import "../vars.scss";

.信息 {
  position: absolute;
  display: flex;
  width: 20%;
  height: 20%;
  padding: 50px 0 0;
  align-items: center;
  align-content: center;
  justify-content: center;
  flex-wrap: wrap;
  background: url("../assets/demon-head.png") center center no-repeat;
  background-size: auto 100%;

  li {
    font-weight: bold;
    width: 100%;
    filter: drop-shadow(0 0 2px rgba(0, 0, 0, 0.7));
    display: flex;
    flex-wrap: wrap;
    justify-content: center;
    text-shadow: 0 2px 1px black, 0 -2px 1px black, 2px 0 1px black,
    -2px 0 1px black;

    span {
      white-space: nowrap;
    }

    .元数据 {
      text-align: center;
      flex-basis: 100%;
      font-family: PiratesBay, sans-serif;
      font-weight: normal;
    }

    svg {
      margin-right: 10px;
    }

    .玩家 {
      color: #00f700;
    }
    .存活 {
      color: #ff4a50;
    }
    .投票 {
      color: #fff;
    }
    .普通村民 {
      color: $townsfolk;
    }
    .外来者 {
      color: $outsider;
    }
    .爪牙 {
      color: $minion;
    }
    .恶魔 {
      color: $demon;
    }
    .旅行者 {
      color: $traveler;
    }
  }

  li.版本 {
    width: 220px;
    height: 200px;
    max-width: 100%;
    max-height: 100%;
    background-position: 0 center;
    background-repeat: no-repeat;
    background-size: 100% auto;
    position: absolute;
    top: -25%;
  }
}
</style>
