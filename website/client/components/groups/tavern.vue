<template lang="pug">
.row
  world-boss-info-modal
  world-boss-rage-modal
  .col-12.col-sm-8.clearfix.standard-page
    .row
      .col-6.title-details
        h1(v-once) {{ $t('welcomeToTavern') }}

    chat(
      :label="$t('tavernChat')",
      :group="group",
      :placeholder="$t('tavernCommunityGuidelinesPlaceholder')",
      @fetchRecentMessages="fetchRecentMessages()"
    )
  .col-12.col-sm-4.sidebar
    .section
      .grassy-meadow-backdrop
        .daniel_front

      .boss-section
        .world-boss(v-if='group && group.quest && group.quest.active', :style="{background: questData.colors.dark, 'border-color': questData.colors.extralight, 'outline-color': questData.colors.light}")
          .corner-decoration(:style="{top: '-2px', right: '-2px'}")
          .corner-decoration(:style="{top: '-2px', left: '-2px'}")
          .corner-decoration(:style="{bottom: '-2px', right: '-2px'}")
          .corner-decoration(:style="{bottom: '-2px', left: '-2px'}")
          .text-center.float-bar.d-flex.align-items-center
            span.diamond
            span.strong.reduce(:style="{background: questData.colors.dark}") {{ $t('worldBossEvent') }}
            span.diamond
          .boss-gradient.pb-3.pt-3
            p.text-center.reduce(:style="{color: questData.colors.extralight}") {{ $t(`${questData.key}ArtCredit`) }}
            .quest-boss(:class="'background_' + questData.key")
              .quest-boss(:class="'quest_' + questData.key")
              .quest-boss(:class="'phobia_' + questData.key", :style="{display: 'none'}")
          .p-3
            .row.d-flex.align-items-center.mb-2
              .col-sm-6
                strong.float-left {{ questData.boss.name() }}
              .col-sm-6
                span.d-flex.float-right
                  .svg-icon.boss-icon(v-html="icons.swordIcon")
                  span.ml-1.reduce(:style="{color: questData.colors.extralight}") {{ $t('pendingDamage', {damage: pendingDamage()}) }}
            .grey-progress-bar.mb-1
              .boss-health-bar(:style="{width: (group.quest.progress.hp / questData.boss.hp) * 100 + '%'}")
            span.d-flex.align-items-center
              .svg-icon.boss-icon(v-html="icons.healthIcon")
              span.reduce.ml-1.pt-1 {{ $t('bossHealth', {currentHealth: bossCurrentHealth(), maxHealth: questData.boss.hp.toLocaleString()}) }}
            .mt-3.mb-2
              strong.mr-1 {{ $t('rageAttack') }}
              span {{ questData.boss.rage.title() }}
            .grey-progress-bar.mb-1
              .boss-health-bar.rage-bar(:style="{width: (group.quest.progress.rage / questData.boss.rage.value) * 100 + '%'}")
            span.d-flex.align-items-center
              .svg-icon.boss-icon(v-html="icons.rageIcon")
              span.reduce.ml-1.pt-1 {{ $t('bossRage', {currentRage: bossCurrentRage(), maxRage: questData.boss.rage.value.toLocaleString()}) }}
            .row.d-flex.align-items-center.mb-2.mt-2
              .col-sm-4.d-flex
                strong.mr-2 {{ $t('rageStrikes') }}
                .svg-icon.boss-icon.information-icon.m-auto(v-html="icons.informationIcon", v-b-tooltip.hover.top="questData.boss.rage.description()")
              .col-sm-8.d-flex.align-items-center.justify-content-center
                .m-auto(@click="showWorldBossRage('seasonalShop')")
                  img.rage-strike(src="~assets/images/world-boss/rage_strike@2x.png", v-if="!group.quest.extra.worldDmg.seasonalShop")
                  img.rage-strike-active(src="~assets/images/world-boss/rage_strike-seasonalShop@2x.png", v-if="group.quest.extra.worldDmg.seasonalShop")
                .m-auto(@click="showWorldBossRage('market')")
                  img.rage-strike(src="~assets/images/world-boss/rage_strike@2x.png", v-if="!group.quest.extra.worldDmg.market")
                  img.rage-strike-active(src="~assets/images/world-boss/rage_strike-market@2x.png", v-if="group.quest.extra.worldDmg.market")
                .m-auto(@click="showWorldBossRage('quests')")
                  img.rage-strike(src="~assets/images/world-boss/rage_strike@2x.png", v-if="!group.quest.extra.worldDmg.quests")
                  img.rage-strike-active(src="~assets/images/world-boss/rage_strike-quests@2x.png", v-if="group.quest.extra.worldDmg.quests")
            .boss-description.p-3(:style="{'border-color': questData.colors.extralight}", @click="sections.worldBoss = !sections.worldBoss")
              strong.float-left {{ $t('worldBossDescription') }}
              .float-right
                .toggle-down(v-if="!sections.worldBoss")
                  .svg-icon.boss-icon(v-html="icons.chevronIcon")
                .toggle-up(v-if="sections.worldBoss")
                  .svg-icon.boss-icon.reverse(v-html="icons.chevronIcon")
            .mt-3(v-if="sections.worldBoss", v-html="questData.notes()")
        // .text-center.mt-4
          .world-boss-info-button(@click="showWorldBossInfo()") {{$t('whatIsWorldBoss') }}

      .sleep.below-header-sections
        strong(v-once) {{ $t('sleepDescription') }}
        ul
          li(v-once) {{ $t('sleepBullet1') }}
          li(v-once) {{ $t('sleepBullet2') }}
          li(v-once) {{ $t('sleepBullet3') }}
          li(v-once) {{ $t('sleepBullet4') }}
        button.btn.btn-secondary.pause-button(v-if='!user.preferences.sleep', @click='toggleSleep()', v-once) {{ $t('pauseDailies') }}
        button.btn.btn-secondary.pause-button(v-if='user.preferences.sleep', @click='toggleSleep()', v-once) {{ $t('unpauseDailies') }}
    .px-3
      sidebar-section(:title="$t('staffAndModerators')")
        .row
          .col-4.staff(v-for='user in staff', :class='{staff: user.type === "Staff", moderator: user.type === "Moderator", bailey: user.name === "It\'s Bailey"}')
            div
              router-link.title(:to="{'name': 'userProfile', 'params': {'userId': user.uuid}}") {{user.name}}
              .svg-icon.staff-icon(v-html="icons.tierStaff", v-if='user.type === "Staff"')
              .svg-icon.mod-icon(v-html="icons.tierMod", v-if='user.type === "Moderator" && user.name !== "It\'s Bailey"')
              .svg-icon.npc-icon(v-html="icons.tierNPC", v-if='user.name === "It\'s Bailey"')
            .type {{user.type}}

      sidebar-section(:title="$t('helpfulLinks')")
        ul
          li
            a(href='', @click.prevent='modForm()') {{ $t('contactForm') }}
          li
           router-link(to='/static/community-guidelines', v-once) {{ $t('communityGuidelinesLink') }}
          li
            router-link(to="/groups/guild/f2db2a7f-13c5-454d-b3ee-ea1f5089e601") {{ $t('lookingForGroup') }}
          li
           router-link(to='/static/faq', v-once) {{ $t('faq') }}
          li
            a(href='', v-html="$t('glossary')")
          li
            a(href='http://habitica.fandom.com/wiki/Habitica_Wiki', v-once) {{ $t('wiki') }}
          li
            a(href='https://oldgods.net/habitrpg/habitrpg_user_data_display.html', v-once) {{ $t('dataDisplayTool') }}
          li
            router-link(to="/groups/guild/a29da26b-37de-4a71-b0c6-48e72a900dac") {{ $t('reportProblem') }}
          li
            a(href='https://trello.com/c/odmhIqyW/440-read-first-table-of-contents', v-once) {{ $t('requestFeature') }}
          li
            a(href='', v-html="$t('communityForum')")
          li
            router-link(to="/groups/guild/5481ccf3-5d2d-48a9-a871-70a7380cee5a") {{ $t('askQuestionGuild') }}

      sidebar-section(:title="$t('playerTiers')")
        .row
          .col-12
            p(v-once) {{ $t('playerTiersDesc') }}
            ul.tier-list
              li.tier1(v-once)
               | {{ $t('tier1') }}
               .svg-icon.tier1-icon(v-html="icons.tier1")
              li.tier2(v-once)
                | {{ $t('tier2') }}
                .svg-icon.tier2-icon(v-html="icons.tier2")
              li.tier3(v-once)
                | {{ $t('tier3') }}
                .svg-icon.tier3-icon(v-html="icons.tier3")
              li.tier4(v-once)
                | {{ $t('tier4') }}
                .svg-icon.tier4-icon(v-html="icons.tier4")
              li.tier5(v-once)
                | {{ $t('tier5') }}
                .svg-icon.tier5-icon(v-html="icons.tier5")
              li.tier6(v-once)
                | {{ $t('tier6') }}
                .svg-icon.tier6-icon(v-html="icons.tier6")
              li.tier7(v-once)
                | {{ $t('tier7') }}
                .svg-icon.tier7-icon(v-html="icons.tier7")
              li.moderator(v-once)
                | {{ $t('tierModerator') }}
                .svg-icon.mod-icon(v-html="icons.tierMod")
              li.staff(v-once)
                | {{ $t('tierStaff') }}
                .svg-icon.staff-icon(v-html="icons.tierStaff")
              li.npc(v-once)
                | {{ $t('tierNPC') }}
                .svg-icon.npc-icon(v-html="icons.tierNPC")
</template>

<style lang='scss' scoped>
  @import '~client/assets/scss/colors.scss';
  @import '~client/assets/scss/variables.scss';

  h1 {
    color: $purple-200;
  }

  .sidebar {
    background-color: $gray-600;
    padding: 0em;

    .below-header-sections {
      padding: 1em 1.75em 1em 1.75em;
    }
  }

  .pause-button {
    background-color: #ffb445 !important;
    color: $white;
    width: 100%;
  }

  .grassy-meadow-backdrop {
    background-image: url('~assets/images/npc/#{$npc_tavern_flavor}/tavern_background.png');
    background-repeat: repeat-x;
    width: 100%;
    height: 246px;
  }

  .daniel_front {
    background-image: url('~assets/images/npc/#{$npc_tavern_flavor}/tavern_npc.png');
    height: 246px;
    width: 471px;
    background-repeat: no-repeat;
    margin: 0 auto;
  }

  .svg-icon {
    width: 10px;
    display: inline-block;
    margin-left: .5em;
  }

  .tier1-icon, .tier2-icon {
    width: 11px;
  }

  .tier5-icon, .tier6-icon {
    width: 8px;
  }

  .tier7-icon {
    width: 12px;
  }

  .mod-icon {
    width: 13px;
  }

  .npc-icon {
    width: 8px;
  }

  .boss-icon {
    width: 16px;
    margin-top: .1em;
    margin-left: 0;
  }

  .boss-icon-large {
    width: 48px;
  }

  .staff {
    margin-bottom: 1em;

    .staff-icon  {
      width: 11px;
    }

    .title {
      color: #6133b4;
      font-weight: bold;
      display: inline-block;
    }
  }

  .tier-list {
    list-style-type: none;
    padding: 0;
    width: 98%;

    li {
      border-radius: 2px;
      background-color: #edecee;
      border: solid 1px #c3c0c7;
      text-align: center;
      padding: 1em;
      margin-bottom: 1em;
      font-weight: bold;
    }

    .tier1 {
      color: #c42870;
    }

    .tier2 {
      color: #b01515;
    }

    .tier3 {
      color: #d70e14;
    }

    .tier4 {
      color: #c24d00;
    }

    .tier5 {
      color: #9e650f;
    }

    .tier6 {
      color: #2b8363;
    }

    .tier7 {
      color: #167e87;
    }

    .tier8, .moderator {
      color: #277eab;
    }

    .tier9, .staff {
      color: #6133b4;
    }

    .npc {
      color: $black;
    }
  }

  .staff .title {
    color: #6133b4;
  }

  .moderator .title {
    color: #277eab;
  }

  .bailey .title {
    color: $black;
  }

  .boss-section {
    padding: 1.75em;
  }

  .world-boss {
    color: $white;
    border-style: solid;
    border-width: 2px;
    outline-style: solid;
    outline-width: 2px;
    margin: 2px;
    position: relative;
  }

  .quest-boss {
    margin: 1em auto;
  }

  .grey-progress-bar {
    width: 100%;
    height: 15px;
    background-color: rgba(255, 255, 255, 0.24);
    border-radius: 2px;
  }

  .boss-health-bar {
    width: 80%;
    height: 15px;
    margin-bottom: .5em;
    border-top-left-radius: 2px;
    border-bottom-left-radius: 2px;
    background-color: #f74e52;
  }

  .boss-health-bar.rage-bar {
    background-color: #ff944c;
  }

  .boss-gradient {
    background-image: linear-gradient(to bottom, #401f2a, #931f4d);
    margin-top: -1.4em;
  }

  .boss-description {
    border-top: 1px solid;
    margin-left: -16px;
    margin-right: -16px;
    padding: .25em 0 0 .25em;
  }

  .float-bar {
    position: relative;
    top: -16px;
    width: 162px;
    height: 28px;
    border-radius: 2px;
    background-color: inherit;
    margin: auto;
  }

  .corner-decoration {
    position: absolute;
    width: 6px;
    height: 6px;
    background-color: inherit;
    border: inherit;
    outline: inherit;
  }

  .reverse {
    transform: rotate(180deg);
  }

  .diamond {
    margin: auto;
    display: inline-block;
    width: 6px;
    height: 6px;
    -webkit-transform: rotate(-45deg);
    transform: rotate(-45deg);
    background-color: #dc4069;
    border: solid 2px #931f4d;
  }

  .reduce {
    font-size: 12px;
  }

  .rage-strike {
    max-width: 50px;
    height: auto;
  }

  .rage-strike-active {
    max-width: 75px;
    height: auto;
    cursor: pointer;
  }

  .world-boss-info-button {
    width: 100%;
    background-color: $gray-500;
    border-radius: 2px;
    font-size: 14px;
    color: $blue-10;
    padding: 1em;
    cursor: pointer;
  }
</style>

<script>
import { mapState } from 'client/libs/store';
import { goToModForm } from 'client/libs/modform';

import { TAVERN_ID } from '../../../common/script/constants';
import worldBossInfoModal from '../world-boss/worldBossInfoModal';
import worldBossRageModal from '../world-boss/worldBossRageModal';
import sidebarSection from '../sidebarSection';
import chat from './chat';


import challengeIcon from 'assets/svg/challenge.svg';
import chevronIcon from 'assets/svg/chevron-red.svg';
import gemIcon from 'assets/svg/gem.svg';
import healthIcon from 'assets/svg/health.svg';
import informationIconRed from 'assets/svg/information-red.svg';
import questBackground from 'assets/svg/quest-background-border.svg';
import rageIcon from 'assets/svg/rage.svg';
import swordIcon from 'assets/svg/sword.svg';

import tier1 from 'assets/svg/tier-1.svg';
import tier2 from 'assets/svg/tier-2.svg';
import tier3 from 'assets/svg/tier-3.svg';
import tier4 from 'assets/svg/tier-4.svg';
import tier5 from 'assets/svg/tier-5.svg';
import tier6 from 'assets/svg/tier-6.svg';
import tier7 from 'assets/svg/tier-7.svg';
import tierMod from 'assets/svg/tier-mod.svg';
import tierNPC from 'assets/svg/tier-npc.svg';
import tierStaff from 'assets/svg/tier-staff.svg';

import quests from 'common/script/content/quests';
import staffList from '../../libs/staffList';

export default {
  components: {
    worldBossInfoModal,
    worldBossRageModal,
    sidebarSection,
    chat,
  },
  data () {
    return {
      groupId: TAVERN_ID,
      icons: Object.freeze({
        challengeIcon,
        chevronIcon,
        gem: gemIcon,
        healthIcon,
        informationIcon: informationIconRed,
        questBackground,
        rageIcon,
        swordIcon,
        tier1,
        tier2,
        tier3,
        tier4,
        tier5,
        tier6,
        tier7,
        tierMod,
        tierNPC,
        tierStaff,
      }),
      group: {
        chat: [],
      },
      sections: {
        worldBoss: true,
      },
      staff: staffList,
    };
  },
  computed: {
    ...mapState({user: 'user.data'}),
    questData () {
      if (!this.group.quest) return {};
      return quests.quests[this.group.quest.key];
    },
  },
  async mounted () {
    this.group = await this.$store.dispatch('guilds:getGroup', {groupId: TAVERN_ID});
  },
  methods: {
    modForm () {
      goToModForm(this.user);
    },
    toggleSleep () {
      this.$store.dispatch('user:sleep');
    },

    pendingDamage () {
      if (!this.user.party.quest.progress.up) return 0;
      return this.$options.filters.floor(this.user.party.quest.progress.up, 10);
      // keep user's pending damage consistent with how it's displayed on the party page
    },
    bossCurrentHealth () {
      if (!this.group.quest.progress.hp) return 0;

      return Math.ceil(parseFloat(this.group.quest.progress.hp)).toLocaleString();
    },
    bossCurrentRage () {
      if (!this.group.quest.progress.hp) return 0;

      return Math.floor(parseFloat(this.group.quest.progress.rage)).toLocaleString();
    },
    showWorldBossInfo () {
      this.$root.$emit('bv::show::modal', 'world-boss-info');
    },
    showWorldBossRage (npc) {
      if (this.group.quest.extra.worldDmg[npc]) {
        this.$store.state.rageModalOptions.npc = npc;
        this.$root.$emit('bv::show::modal', 'world-boss-rage');
      }
    },
    async fetchRecentMessages () {
      this.group = await this.$store.dispatch('guilds:getGroup', {groupId: TAVERN_ID});
    },
  },
};
</script>
