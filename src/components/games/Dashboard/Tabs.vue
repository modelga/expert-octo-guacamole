<template>
  <div class="notification">
    <b-tabs v-model="tab">
      <b-tab-item label="Table" v-if="game.table && game.table.length > 0" icon="list-ol">
        <GameTable :game="game" v-if="!isMobile" />
        <FocusedTable :game="game" v-else :count="game.table.length*2" :focus="id" />
      </b-tab-item>
      <b-tab-item label="Schedule" v-if="game.status !== 'OPEN'" icon="calendar">
        <Schedule :game="game" @updated="(game) => $emit('updated',game)" @needFocus="focus(1)" />
      </b-tab-item>
      <b-tab-item label="Results" v-if="hasResults" icon="calendar">
        <Results :game="game" @updated="(game) => $emit('updated',game)" @needFocus="focus(1)" />
      </b-tab-item>
      <b-tab-item label="Competitors" icon="users">
        <Competitors :game="game" @updated="(game) => $emit('updated',game)" />
      </b-tab-item>
      <b-tab-item label="Timeline" icon="clock-o" v-if="isCompetitor">
        <Timeline :game="game" />
      </b-tab-item>
      <b-tab-item label="Rules" v-if="(game.rules || '').length" icon="list-ul">
        <Rules :game="game" />
      </b-tab-item>
      <b-tab-item label="Settings" v-if="isAdmin" icon="cog">
        <Settings :game="game" @updated="(game) => $emit('updated',game)" />
      </b-tab-item>
    </b-tabs>
  </div>
</template>
<script>
import { mapGetters } from 'vuex';
import * as R from 'ramda';
import Schedule from './Schedule';
import GameTable from './Table';
import Competitors from './Competitors';
import FocusedTable from '../FocusedTable';
import Settings from './Settings';
import Results from './Results';
import Timeline from './timeline/Timeline';
import Rules from './Rules';

export default {
  name: 'GameTabs',
  components: {
    Schedule, GameTable, FocusedTable, Competitors, Settings, Rules, Timeline, Results,
  },
  props: {
    game: { type: Object, required: true },
  },
  data() { return { tab: 0 }; },
  computed: {
    ...mapGetters(['id', 'isAdmin', 'isMobile']),
    isCompetitor() {
      return this.game.table.map(t => t.id).indexOf(this.id) !== -1;
    },
    hasResults() {
      return R.pipe(R.filter(R.complement(R.propEq('status', 'SCHEDULED'))), R.values, R.isEmpty, R.not)(this.game.schedule);
    },
  },
  methods: { focus(tab) { this.tab = tab; } },
};
</script>
