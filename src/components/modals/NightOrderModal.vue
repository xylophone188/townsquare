<template>
  <Modal
    class="night-reference"
    @close="toggleModal('nightOrder')"
    v-if="modals.nightOrder && roles.size"
  >
    <font-awesome-icon
      @click="toggleModal('reference')"
      icon="address-card"
      class="toggle"
      title="显示角色参考"
    />
    <h3>
      夜晚顺序
      <font-awesome-icon icon="cloud-moon" />
      {{ edition.name || "自定义脚本" }}
    </h3>
    <div class="night">
      <ul class="first">
        <li class="headline">首晚</li>
        <li
          v-for="role in rolesFirstNight"
          :key="role.name"
          :class="[role.team]"
        >
          <span class="name">
            {{ role.name }}
            <span class="player" v-if="role.players.length">
              <br />
              <small
                v-for="(player, index) in role.players"
                :class="{ dead: player.isDead }"
                :key="index"
                >{{
                  player.name + (role.players.length > index + 1 ? "," : "")
                }}</small
              >
            </span>
          </span>
          <span
            class="icon"
            v-if="role.id"
            :style="{
              backgroundImage: `url(${
                role.image && grimoire.isImageOptIn
                  ? role.image
                  : require('../../assets/icons/' +
                      (role.imageAlt || role.id) +
                      '.png')
              })`
            }"
          ></span>
          <span class="reminder" v-if="role.firstNightReminder">
            {{ role.firstNightReminder }}
          </span>
        </li>
      </ul>
      <ul class="other">
        <li class="headline">其他夜晚</li>
        <li
          v-for="role in rolesOtherNight"
          :key="role.name"
          :class="[role.team]"
        >
          <span
            class="icon"
            v-if="role.id"
            :style="{
              backgroundImage: `url(${
                role.image && grimoire.isImageOptIn
                  ? role.image
                  : require('../../assets/icons/' +
                      (role.imageAlt || role.id) +
                      '.png')
              })`
            }"
          ></span>
          <span class="name">
            {{ role.name }}
            <span class="player" v-if="role.players.length">
              <br />
              <small
                v-for="(player, index) in role.players"
                :class="{ dead: player.isDead }"
                :key="index"
                >{{
                  player.name + (role.players.length > index + 1 ? "," : "")
                }}</small
              >
            </span>
          </span>
          <span class="reminder" v-if="role.otherNightReminder">
            {{ role.otherNightReminder }}
          </span>
        </li>
      </ul>
    </div>
  </Modal>
</template>

<script>
import Modal from "./Modal";
import { mapMutations, mapState } from "vuex";

export default {
  components: {
    Modal
  },
  computed: {
    rolesFirstNight: function() {
      const rolesFirstNight = [];
      // add minion / demon infos to night order sheet
      if (this.players.length > 6) {
        rolesFirstNight.push(
          {
            id: "evil",
            name: "爪牙信息",
            firstNight: 5,
            team: "爪牙",
            players: this.players.filter(p => p.role.team === "爪牙"),
            firstNightReminder:
              "• 如果有多个爪牙，请它们相互眼神交流。 • 展示“这是魔鬼”卡片。指向魔鬼。"
          },
          {
            id: "evil",
            name: "魔鬼信息与虚张声势",
            firstNight: 8,
            team: "魔鬼",
            players: this.players.filter(p => p.role.team === "魔鬼"),
            firstNightReminder:
              "• 展示“这些是你的爪牙”卡片。指向每个爪牙。 • 展示“这些角色不在游戏中”卡片。展示三张未在游戏中的善良角色的角色代币。"
          }
        );
      }
      this.roles.forEach(role => {
        const players = this.players.filter(p => p.role.id === role.id);
        if (role.firstNight && (role.team !== "旅行者" || players.length)) {
          rolesFirstNight.push(Object.assign({ players }, role));
        }
      });
      this.fabled
        .filter(({ firstNight }) => firstNight)
        .forEach(fabled => {
          rolesFirstNight.push(Object.assign({ players: [] }, fabled));
        });
      rolesFirstNight.sort((a, b) => a.firstNight - b.firstNight);
      return rolesFirstNight;
    },
    rolesOtherNight: function() {
      const rolesOtherNight = [];
      this.roles.forEach(role => {
        const players = this.players.filter(p => p.role.id === role.id);
        if (role.otherNight && (role.team !== "旅行者" || players.length)) {
          rolesOtherNight.push(Object.assign({ players }, role));
        }
      });
      this.fabled
        .filter(({ otherNight }) => otherNight)
        .forEach(fabled => {
          rolesOtherNight.push(Object.assign({ players: [] }, fabled));
        });
      rolesOtherNight.sort((a, b) => a.otherNight - b.otherNight);
      return rolesOtherNight;
    },
    ...mapState(["roles", "modals", "edition", "grimoire"]),
    ...mapState("players", ["players", "fabled"])
  },
  methods: {
    ...mapMutations(["toggleModal"])
  }
};
</script>

<style lang="scss" scoped>
@import "../../vars.scss";

.toggle {
  position: absolute;
  left: 20px;
  top: 15px;
  cursor: pointer;
  &:hover {
    color: red;
  }
}

h3 {
  margin: 0 40px;
  svg {
    vertical-align: middle;
  }
}

h4 {
  text-transform: capitalize;
  display: flex;
  align-items: center;
  height: 20px;
  &:before,
  &:after {
    content: " ";
    width: 100%;
    height: 1px;
    border-radius: 2px;
  }
  &:before {
    margin-right: 15px;
  }
  &:after {
    margin-left: 15px;
  }
}

.fabled {
  .name {
    background: linear-gradient(90deg, $fabled, transparent 35%);
    .night .other & {
      background: linear-gradient(-90deg, $fabled, transparent 35%);
    }
  }
}
.townsfolk {
  .name {
    background: linear-gradient(90deg, $townsfolk, transparent 35%);
    .night .other & {
      background: linear-gradient(-90deg, $townsfolk, transparent 35%);
    }
  }
}
.outsider {
  .name {
    background: linear-gradient(90deg, $outsider, transparent 35%);
    .night .other & {
      background: linear-gradient(-90deg, $outsider, transparent 35%);
    }
  }
}
.minion {
  .name {
    background: linear-gradient(90deg, $minion, transparent 35%);
    .night .other & {
      background: linear-gradient(-90deg, $minion, transparent 35%);
    }
  }
}
.demon {
  .name {
    background: linear-gradient(90deg, $demon, transparent 35%);
    .night .other & {
      background: linear-gradient(-90deg, $demon, transparent 35%);
    }
  }
}
ul {
  li {
    display: flex;
    width: 100%;
    margin-bottom: 3px;
    .icon {
      width: 6vh;
      background-size: cover;
      background-position: 0 0;
      flex-grow: 0;
      flex-shrink: 0;
      text-align: center;
      margin: 0 2px;
      &:after {
        content: " ";
        display: block;
        padding-top: 66%;
      }
    }
    .name {
      flex-grow: 0;
      flex-shrink: 0;
      width: 15%;
      text-align: right;
      font-size: 110%;
      padding: 5px;
      border-left: 1px solid rgba(255, 255, 255, 0.4);
      border-right: 1px solid rgba(255, 255, 255, 0.4);
      small {
        color: #888;
        margin-right: 5px;
        &.dead {
          text-decoration: line-through;
        }
      }
    }
    .reminder {
      position: fixed;
      padding: 5px 10px;
      left: 50%;
      bottom: 10%;
      width: 500px;
      z-index: 25;
      background: rgba(0, 0, 0, 0.75);
      border-radius: 10px;
      border: 3px solid black;
      filter: drop-shadow(0 4px 6px rgba(0, 0, 0, 0.5));
      text-align: left;
      pointer-events: none;
      opacity: 0;
      transition: opacity 200ms ease-in-out;
      margin-left: -250px;
    }
    &:hover .reminder {
      opacity: 1;
    }
  }
  &.legend {
    font-weight: bold;
    height: 20px;
    margin-top: 10px;
    li span {
      background: none;
      height: auto;
      font-family: inherit;
      font-size: inherit;
    }
    .icon:after {
      padding-top: 0;
    }
  }
}

.night {
  display: flex;
  align-items: flex-start;
  justify-content: center;
  > *:first-child {
    margin-right: 2vh;
  }
  > * {
    flex-grow: 0;
    flex-wrap: nowrap;
    flex-direction: column;
  }
  .headline {
    display: block;
    font-weight: bold;
    border-bottom: 1px solid rgba(255, 255, 255, 0.4);
    padding: 5px 10px;
    border-radius: 0;
    text-align: center;
  }
  .name {
    flex-grow: 1;
  }
  .first {
    .name {
      border-left: 0;
    }
  }
  .other {
    li .name {
      text-align: left;
      border-right: 0;
    }
  }
}

/** 当广场设置为“公共”时隐藏玩家 **/
#townsquare.public ~ .night-reference .modal .player {
  display: none;
}
</style>
