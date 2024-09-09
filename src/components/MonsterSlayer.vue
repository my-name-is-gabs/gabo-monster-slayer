<script>
const healthBarBg = (barValue) =>
  barValue > 70 ? "#05D784" : barValue > 40 ? "#F8C301" : "#BF0100";

export default {
  data() {
    return {
      gameLog: [],
      PlayerHealth: 100,
      monsterHealth: 100,
      showInGameActionCenter: false,
      healCount: 3,
      attackCount: 0,
      damageDealt: 0,
      damageTaken: 0,
      hmSpecialAttacks: 0,
    };
  },
  methods: {
    rand(min, max) {
      return Math.floor(Math.random() * (max - min + 1) + min);
    },
    startNewGame() {
      //setting values to default
      this.resetValues();

      //toggling action center
      this.showInGameActionCenter = true;
      //clearing game log
      this.gameLog = [];
    },
    attackMonster() {
      //generate a random attack damage
      let damage = this.rand(1, 10);

      //decrease monster health
      if (this.monsterHealth - damage > 0) {
        this.monsterHealth -= damage;
      } else {
        this.monsterHealth = 0;
      }

      //============stats==========
      //increase attack count in game log
      this.attackCount += 1;
      //increase damage dealt
      this.damageDealt += damage;
      //log the stats
      this.logAction("attack", "player", damage);

      //make monster attack me with a greater random value
      this.recieveDamage(6, 10);
    },
    specialAttackMonster() {
      //generate a random damage and increase it
      //in case the player hasn't used a special
      //attacks for 2 times

      let damage = this.attackCount == 8 ? this.rand(8, 10) : this.rand(6, 9);

      //decrease monester health
      if (this.monsterHealth - damage > 0) {
        this.monsterHealth -= damage;
      } else {
        this.monsterHealth = 0;
      }

      this.attackCount -= 4;

      this.damageDealt += damage;

      this.hmSpecialAttacks++;

      //log to the states
      this.logAction("specialAttack", "Player", damage);
    },
    healPlayer() {
      let heal = this.rand(5, 10);

      this.PlayerHealth += heal;

      this.healCount -= 1;

      this.damageTaken -= heal;

      this.logAction("heal", "Player", heal);
    },
    surrender() {
      const willSurrender = window.confirm(
        "Are you sure you want to surrender?"
      );
      if (willSurrender) {
        this.resetValues();
        this.showInGameActionCenter = false;
      }
    },
    recieveDamage(min, max) {
      let damage = this.rand(min, max);

      if (this.PlayerHealth - damage > 0) {
        this.PlayerHealth -= damage;
      } else {
        this.PlayerHealth = 0;
      }

      this.damageTaken += damage;

      this.logAction("attacked", "monster", damage);
    },
    logAction(actionTaken, entity, value) {
      let action = {
        actionTaken: actionTaken,
        entity: entity,
        actionValue: value,
      };

      this.gameLog.push(action);
    },
    resetValues() {
      this.PlayerHealth = 100;
      this.monsterHealth = 100;
      this.attackCount = 0;
      this.damageDealt = 0;
      this.damageTaken = 0;
      this.healCount = 3;
    },
  },
  computed: {
    stylePlayerHealthBar: function () {
      return {
        width: this.PlayerHealth + "%",
        backgroundColor: healthBarBg(this.PlayerHealth),
        height: "30px",
      };
    },

    styleMonsterHealthBar: function () {
      return {
        width: this.monsterHealth + "%",
        backgroundColor: healthBarBg(this.monsterHealth),
        height: "30px",
      };
    },

    healsUsed: function () {
      return 3 - this.healCount;
    },
    anyHealthZeroed: function () {
      return this.PlayerHealth == 0 || this.monsterHealth == 0;
    },
  },
};
</script>

<template>
  <header>
    <h1 class="game-title">Monster Slayer</h1>
  </header>
  <main class="container my-4" id="app-container">
    <div class="container row">
      <div class="health-bar-container col-md-6">
        <div v-if="monsterHealth === 0 && PlayerHealth > 0">
          <img
            class="img-fluid player-won"
            src="../assets/player-f2.gif"
            alt="Player winner"
          />
        </div>
        <div v-if="PlayerHealth !== 0 && monsterHealth !== 0">
          <img
            class="img-fluid player-image"
            src="../assets/player-f1.gif"
            alt="Player Poker face"
          />
        </div>
        <div v-if="PlayerHealth === 0">
          <img
            class="img-fluid player-image"
            src="../assets/dead.png"
            alt="Player dead"
          />
        </div>

        <div class="container d-flex flex-column">
          <span class="health">{{ PlayerHealth }}\100</span>
          <div class="health-bar">
            <span :style="stylePlayerHealthBar"></span>
          </div>
        </div>
      </div>
      <div class="health-bar-container col-md-6">
        <div v-if="monsterHealth > 70 || PlayerHealth === 0">
          <img
            class="img-fluid monster-image"
            src="../assets/mons_f1.png"
            alt="monster face 1"
          />
        </div>
        <div v-else-if="monsterHealth > 50">
          <img
            class="img-fluid monster-image"
            src="../assets/mons_f2.png"
            alt="monster face 2"
          />
        </div>
        <div v-else-if="monsterHealth > 25">
          <img
            class="img-fluid monster-image"
            src="../assets/mons_f3.png"
            alt="monster face 3"
          />
        </div>
        <div v-else-if="monsterHealth > 0">
          <img
            class="img-fluid monster-image"
            src="../assets/mons_f4.png"
            alt="monster face 4"
          />
        </div>
        <div v-else>
          <img
            class="img-fluid monster-image"
            src="../assets/dead.png"
            alt="monster dead"
          />
        </div>
        <div class="container d-flex flex-column">
          <span class="health">{{ monsterHealth }}\100</span>
          <div class="health-bar">
            <span :style="styleMonsterHealthBar"></span>
          </div>
        </div>
      </div>
    </div>
    <div class="action-center">
      <div class="new-game" v-if="!showInGameActionCenter || anyHealthZeroed">
        <button class="action-btn-start" @click="startNewGame">new game</button>
      </div>
      <div class="in-game" v-else>
        <button class="action-btn" @click="attackMonster">Attack</button>
        <button
          class="action-btn"
          :disabled="attackCount < 4"
          @click="specialAttackMonster"
        >
          Special Skill
        </button>
        <button
          class="action-btn"
          :disabled="PlayerHealth > 50 || healCount < 1"
          @click="healPlayer"
        >
          Heal
        </button>

        <button class="action-btn" @click="surrender">Surrender</button>
      </div>
    </div>
    <div class="game-log-container">
      <div class="header flex-center">
        <h2>Game log</h2>
      </div>
      <div class="game-log">
        <ul>
          <li v-for="log in gameLog">
            <span class="log red" v-if="log.actionTaken == 'attacked'"
              >The {{ log.entity }} attacked the Player, dealing
              {{ log.actionValue }} damage.</span
            >
            <span class="log green" v-if="log.actionTaken == 'attack'"
              >The {{ log.entity }} attacked the Monster, dealing
              {{ log.actionValue }} damage.</span
            >
            <span class="log yellow" v-if="log.actionTaken == 'specialAttack'"
              >The {{ log.entity }} launched a Special Attack on the Monster,
              dealing {{ log.actionValue }} damage.</span
            >
            <span class="log blue" v-if="log.actionTaken == 'heal'"
              >The {{ log.entity }} performed a heal, gaining
              {{ log.actionValue }} health.</span
            >
          </li>
        </ul>
      </div>
      <div class="game-stats">
        <span class="yellow">
          Special Attacks Used: {{ hmSpecialAttacks }}
        </span>
        <span class="green"> Total Damage Dealt: {{ damageDealt }} </span>
        <span class="red"> Total Damage Taken: {{ damageTaken }} </span>
        <span class="blue"> Heals Used: {{ healsUsed }} </span>
      </div>
    </div>
  </main>
</template>

<style scoped>
header {
  align-items: flex-end !important;
  gap: 10px;
  background-color: #c83b50;
  text-align: center;
  padding: 1rem;
  color: #fff;
  box-shadow: 0 5px 10px rgba(0, 0, 0, 0.2);
}

#app-container {
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  gap: 3.2rem;
}

.game-title {
  font-family: "Press Start 2P", "Courier New", Courier, monospace;
  letter-spacing: 3px;
}

span {
  display: block;
}

.player-image {
  width: 250px;
}

.player-won {
  width: 400px;
}

.monster-image {
  width: 250px;
}

.health-bar-container {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  align-items: center;
  width: 100%;
  gap: 12px;
}
.health-bar-container span {
  font-size: 130%;
}
.health-bar-container .health-bar {
  width: 100%;
  height: 30px;
  background-color: #999;
}
.health-bar-container .health-bar span {
  display: block;
  height: 100%;
  transition: width 100ms ease-in;
}
.action-center .new-game,
.action-center .in-game {
  gap: 1rem;
}

.action-btn,
.action-btn-start {
  outline: none;
  cursor: pointer;
  padding: 10px 24px;
  font-size: 2rem;
  font-family: inherit;
  box-shadow: 4px 4px 0px #7b2230;
  border: 2px solid #7b2230;
  color: #333;
}

.action-btn-start {
  background-color: #e65c19;
  color: white;
  font-family: "Press Start 2P", "Courier New", Courier, monospace;
}

.action-btn {
  background-color: #fff;
}

.action-btn:active {
  background-color: #c7c7c7;
}
.action-btn:disabled {
  color: white;
  cursor: not-allowed;
  box-shadow: none;
  border-color: transparent;
  opacity: 0.5;
  background-color: transparent;
}

.flex-center,
header,
.action-center .new-game,
.action-center .in-game,
.game-log-container .game-stats,
footer {
  display: flex;
  justify-content: center;
  align-items: center;
  flex-wrap: wrap;
}

.img-btn {
  width: 58px;
  height: 58px;
}

.img-player {
  width: 80px;
  height: 80px;
}

.game-log-container {
  text-align: center;
  border: 1px solid #dfdfdf;
}
.game-log-container .header {
  padding: 0.5rem 0;
  background-color: #f7f7f7;
  border-bottom: 1px solid #dfdfdf;
  color: #606060;
}
.game-log-container .header h2 {
  letter-spacing: 3px;
}
.game-log-container .game-log {
  height: 150px;
  max-height: 150px;
  overflow: auto;
}
.game-log-container .game-log .log {
  padding: 5px 0;
}
.game-log-container .game-stats {
  background-color: #f7f7f7;
  gap: 0.5rem;
  border-top: 1px solid #dfdfdf;
  padding: 1rem 2rem;
}
.game-log-container .game-stats span {
  flex-grow: 1;
  padding: 5px 10px;
}
.game-log-container .yellow {
  color: #856404;
  background-color: #fff3cd;
}
.game-log-container .green {
  color: #155724;
  background-color: #d4edda;
}
.game-log-container .red {
  color: #721c24;
  background-color: #f8d7da;
}
.game-log-container .blue {
  color: #004085;
  background-color: #cce5ff;
}
</style>
