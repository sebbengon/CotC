<template>
<div>
  <Modal class="characters" @close="toggleModal('reference')" v-if="modals.reference && roles.size">
    <font-awesome-icon @click="toggleModal('nightOrder')" icon="cloud-moon" class="toggle" title="Show Night Order" />
    <font-awesome-icon @click="printReference" icon="print" class="toggle print" title="Skriv ut" />
    <h3>
      Karaktärsreferens
      <font-awesome-icon icon="address-card" />
      {{ edition.name || 'Custom Script' }}
    </h3>
    <div v-for="(teamRoles, team) in rolesGrouped" :key="team" :class="['team', team]">
      <aside>
        <!-- <h4>changeTeamNames</h4> -->
        <h4>
          {{
            team === 'townsfolk'
              ? 'Stadsbo'
              : team === 'outsider'
              ? 'Lantbo'
              : team === 'minion'
              ? 'Kumpan'
              : team === 'demon'
              ? 'HF'
              : ''
          }}
        </h4>
      </aside>
      <ul>
        <li v-for="role in teamRoles" :class="[team]" :key="role.id">
          <span
            class="icon"
            v-if="role.id"
            :style="{
              backgroundImage: `url(${
                role.image && grimoire.isImageOptIn
                  ? role.image
                  : require('../../assets/icons/' + (role.imageAlt || role.id) + '.png')
              })`,
            }"
          ></span>
          <div class="role">
            <span class="player" v-if="Object.keys(playersByRole).length">{{
              playersByRole[role.id] ? playersByRole[role.id].join(', ') : ''
            }}</span>
            <span class="name">{{ role.name }}</span>
            <span class="ability">{{ role.ability }}</span>
          </div>
        </li>
        <li :class="[team]"></li>
        <li :class="[team]"></li>
      </ul>
    </div>

    <div class="team jinxed" v-if="jinxed.length">
      <aside>
        <h4>Ändringar</h4>
      </aside>
      <ul>
        <li class="jinx" v-for="(jinx, index) in jinxed" :key="index">
          <span
            class="icon"
            :style="{
              backgroundImage: `url(${require('../../assets/icons/' + jinx.first.id + '.png')})`,
            }"
          ></span>
          <span
            class="icon"
            :style="{
              backgroundImage: `url(${require('../../assets/icons/' + jinx.second.id + '.png')})`,
            }"
          ></span>
          <div class="role">
            <span class="name">{{ jinx.first.name }} & {{ jinx.second.name }}</span>
            <span class="ability">{{ jinx.reason }}</span>
          </div>
        </li>
        <li></li>
        <li></li>
      </ul>
    </div>
  </Modal>
<div class="print-sheet">
  <h1>Karaktärsreferens – {{ edition.name || 'Custom Script' }}</h1>

  <div class="columns">
    <div
      v-for="(teamRoles, team) in rolesGrouped"
      :key="team"
      class="team"
      :class="team"
    >
      <h2>
        {{
          team === 'townsfolk'
            ? 'Stadsbo'
            : team === 'outsider'
            ? 'Lantbo'
            : team === 'minion'
            ? 'Kumpan'
            : team === 'demon'
            ? 'HF'
            : ''
        }}
      </h2>

      <div
        v-for="role in teamRoles"
        :key="role.id"
        class="role-row"
      >
        <img
          class="icon"
          :src="
            role.image && grimoire.isImageOptIn
              ? role.image
              : require('../../assets/icons/' + (role.imageAlt || role.id) + '.png')
          "
        />

        <div class="text">
          <div class="name">{{ role.name }}</div>
          <div class="ability">{{ role.ability }}</div>
        </div>
      </div>
    </div>
  </div>

  <!-- JINX PAGE -->
  <div class="jinx-page" v-if="jinxed.length">
    <h2>Ändringar</h2>

    <div v-for="(jinx, i) in jinxed" :key="i" class="jinx-row">
      <img :src="require('../../assets/icons/' + jinx.first.id + '.png')" />
      <img :src="require('../../assets/icons/' + jinx.second.id + '.png')" />

      <div>
        <strong>{{ jinx.first.name }} & {{ jinx.second.name }}</strong>
        <div>{{ jinx.reason }}</div>
      </div>
    </div>
  </div>
</div>
</div>
</template>

<script>
import Modal from './Modal'
import { mapMutations, mapState } from 'vuex'

export default {
  components: {
    Modal,
  },
  computed: {
    /**
     * Return a list of jinxes in the form of role IDs and a reason
     * @returns {*[]} [{first, second, reason}]
     */
    jinxed: function() {
      const jinxed = []
      this.roles.forEach((role) => {
        if (this.jinxes.get(role.id)) {
          this.jinxes.get(role.id).forEach((reason, second) => {
            if (this.roles.get(second)) {
              jinxed.push({
                first: role,
                second: this.roles.get(second),
                reason,
              })
            }
          })
        }
      })
      return jinxed
    },
    rolesGrouped: function() {
      const rolesGrouped = {}
      this.roles.forEach((role) => {
        if (!rolesGrouped[role.team]) {
          rolesGrouped[role.team] = []
        }
        rolesGrouped[role.team].push(role)
      })
      delete rolesGrouped['traveler']
      return rolesGrouped
    },
    playersByRole: function() {
      const players = {}
      this.players.forEach(({ name, role }) => {
        if (role && role.id && role.team !== 'traveler') {
          if (!players[role.id]) {
            players[role.id] = []
          }
          players[role.id].push(name)
        }
      })
      return players
    },
    ...mapState(['roles', 'modals', 'edition', 'grimoire', 'jinxes']),
    ...mapState('players', ['players']),
  },
  methods: {
    ...mapMutations(['toggleModal']),
    printReference() {
      window.print()
  },
  }
}
</script>

<style lang="scss" scoped>
@import '../../vars.scss';

.toggle {
  position: absolute;
  left: 20px;
  top: 15px;
  cursor: pointer;
  z-index: 100;
  &:hover {
    color: red;
  }
}
.toggle.print {
  left: 55px; /* flytta den åt höger */
  }

h3 {
  margin: 0 40px;
  svg {
    vertical-align: middle;
  }
}

.townsfolk {
  .name {
    color: $townsfolk;
  }
  aside {
    background: linear-gradient(-90deg, $townsfolk, transparent);
  }
}
.outsider {
  .name {
    color: $outsider;
  }
  aside {
    background: linear-gradient(-90deg, $outsider, transparent);
  }
}
.minion {
  .name {
    color: $minion;
  }
  aside {
    background: linear-gradient(-90deg, $minion, transparent);
  }
}
.demon {
  .name {
    color: $demon;
  }
  aside {
    background: linear-gradient(-90deg, $demon, transparent);
  }
}

.jinxed {
  .name {
    color: $fabled;
  }
  aside {
    background: linear-gradient(-90deg, $fabled, transparent);
  }
}

.team {
  display: flex;
  align-items: stretch;
  width: 100%;
  &:not(:last-child):after {
    content: ' ';
    display: block;
    width: 25%;
    height: 1px;
    background: linear-gradient(90deg, #000000, transparent);
    position: absolute;
    left: 0;
    bottom: 0;
  }
  aside {
    width: 30px;
    display: flex;
    flex-grow: 0;
    flex-shrink: 0;
    align-items: center;
    justify-content: center;
    align-content: center;
    overflow: hidden;
    text-shadow: 0 0 4px black;
  }

  h4 {
    text-align: center;
    transform: rotate(90deg);
    transform-origin: center;
    font-size: 80%;
    color: white;
  }

  &.jinxed {
    .icon {
      margin: 0 -5px;
    }
  }
}

ul {
  flex-grow: 1;
  display: flex;
  padding: 5px 0;

  li {
    display: flex;
    align-items: center;
    flex-grow: 1;
    width: 420px;
    margin: 5px 0;
    .icon {
      width: 8vh;
      background-size: contain;
      flex-shrink: 0;
      flex-grow: 0;
      height: 60px;
      background-position: center;
      background-repeat: no-repeat;
      margin: 5px;
      &:after {
        content: ' ';
        display: block;
        padding-top: 75%;
      }
    }
    .role {
      line-height: 80%;
      flex-grow: 1;
    }
    .name {
      font-weight: bold;
      font-size: 75%;
      display: block;
    }
    .player {
      color: #888;
      float: right;
      font-size: 60%;
    }
    .ability {
      font-size: 70%;
    }
  }
}

.jinx {
  display: grid;
  grid-template-columns: max-content max-content auto;
  gap: 0.5rem;
  margin-left: 10px;
}

/** break into 1 column below 1200px **/
@media screen and (max-width: 1199.98px) {
  .modal {
    max-width: 60%;
  }
  ul {
    li {
      .icon {
        width: 6vh;
      }
      .role {
        line-height: 100%;
      }
      .name {
        font-size: 100%;
      }
      .player {
        font-size: 100%;
      }
      .ability {
        font-size: 90%;
      }
    }
  }
}

/** trim icon size on maximized one-column sheet **/
@media screen and (max-width: 991.98px) {
  .characters .modal.maximized ul li .icon {
    width: 5.1vh;
  }
}

/** hide players when town square is set to "public" **/
#townsquare.public ~ .characters .modal .player {
  display: none;
}
/* ============================================
   PRINT LAYOUT — A4, två kolumner, jinx separat
   ============================================ */
.print-sheet {
  display: none;
}

@media print {
  @page {
    size: A4;
    margin: 12mm;
  }

  body * {
    visibility: hidden;
  }

  .print-sheet,
  .print-sheet * {
    visibility: visible;
  }

  .print-sheet {
    display: block;
    position: absolute;
    top: 0;
    left: 0;
    width: 210mm;
  }

  /* ===== TYPO ===== */
  h1 {
    font-size: 18pt;
    margin-bottom: 10mm;
  }

  h2 {
    font-size: 12pt;
    margin-top: 6mm;
    margin-bottom: 3mm;
  }

  /* ===== TWO COLUMNS ===== */
  .columns {
    column-count: 2;
    column-gap: 12mm;
  }

  .team {
    break-inside: avoid;
    margin-bottom: 6mm;
  }

  /* ===== ROLE ===== */
  .role-row {
    display: flex;
    gap: 6px;
    margin-bottom: 4px;
    break-inside: avoid;
  }

  .icon {
    width: 18px;
    height: 18px;
    object-fit: contain;
  }

  .name {
    font-weight: bold;
    font-size: 9pt;
  }

  .ability {
    font-size: 8pt;
  }

  /* ===== JINX PAGE ===== */
  .jinx-page {
    page-break-before: always;
  }

  .jinx-row {
    display: flex;
    gap: 6px;
    margin-bottom: 6px;
  }

  .jinx-row img {
    width: 18px;
    height: 18px;
  }
}

</style>
