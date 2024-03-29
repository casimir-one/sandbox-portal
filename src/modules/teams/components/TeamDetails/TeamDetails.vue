<template>
  <div v-if="ready">
    <v-toolbar v-if="canEdit">
      <v-spacer />
      <ve-stack
        flow="column"
        :gap="4"
        class="justify-end"
      >
        <v-btn
          text
          color="primary"
          small
          :to="{name: 'teams.edit'}"
          class="align-self-center"
        >
          <v-icon left>
            mdi-pencil-outline
          </v-icon>
          {{ $t('teams.details.edit') }}
        </v-btn>

        <v-btn
          text
          color="primary"
          small
          :to="{name: 'nftCollections.create', query: { teamId }}"
          class="align-self-center"
        >
          {{ $t('teams.details.createNftCollection') }}
        </v-btn>
      </ve-stack>
    </v-toolbar>

    <c-team-details
      v-if="schema"
      :team="team"
      :schema="schema"
    />
    <v-divider />

    <vex-section>
      <ve-stack :gap="24">
        <div class="d-flex justify-space-between">
          <span class="text-h6">{{ $t('teams.details.members') }}:</span>
          <v-btn
            v-if="canEdit"
            text
            color="primary"
            mb-4
            small
            @click.stop="handleInviteClick"
          >
            {{ $t('teams.details.inviteMembers') }}
          </v-btn>
        </div>

        <team-add-member-modal
          v-model="dialog"
          :team-id="teamId"
          :team-members="team.members"
          @success="handleAddMemberSuccess"
          @error="handleError"
        />

        <team-member-cards
          :users="team.members"
          :team-id="teamId"
          :can-edit="canEdit"
          @success="handleRemoveMemberSuccess"
          @error="handleError"
        />
      </ve-stack>
    </vex-section>

    <v-divider />

    <vex-section>
      <ve-stack :gap="24">
        <span class="text-h6">{{ $t('teams.details.nftCollections') }}:</span>
        <nft-collection-cards-list :team-id="teamId" />
      </ve-stack>
    </vex-section>
  </div>
</template>

<script>
  import { VeStack } from '@deip/vue-elements';
  import { VexSection } from '@deip/vuetify-extended';
  import { rolesFactory } from '@/mixins';
  import {
    TeamDetails as CTeamDetails,
    TeamAddMemberModal,
    TeamMemberCards
  } from '@deip/teams-module';
  import { NftCollectionCardsList }
    from '@/modules/nftCollections/components/NftCollectionCardsList';

  export default {
    name: 'TeamDetails',

    components: {
      VeStack,
      VexSection,
      TeamMemberCards,
      TeamAddMemberModal,
      CTeamDetails,
      NftCollectionCardsList
    },

    mixins: [rolesFactory('teamId')],

    props: {
      teamId: {
        type: String,
        default: null
      }
    },

    data() {
      return {
        ready: false,
        dialog: false
      };
    },

    computed: {
      team() {
        return this.teamId ? this.$store.getters['teams/one'](this.teamId) : {};
      },

      schema() {
        return this.$layouts.getMappedData('team.details')?.value;
      },

      canEdit() {
        return this.$$isTeamAdmin;
      }
    },

    created() {
      this.getTeam();
    },

    methods: {
      async getTeam() {
        if (this.teamId) {
          try {
            await this.$store.dispatch('teams/getOne', this.teamId);
          } catch (error) {
            console.error(error);
          }
        }
        this.ready = true;
      },
      handleInviteClick() {
        this.dialog = true;
      },
      handleAddMemberSuccess() {
        this.$notifier.showSuccess(this.$t('teams.cards.successAdd'));
      },
      handleRemoveMemberSuccess() {
        this.$notifier.showSuccess(this.$t('teams.cards.successRemove'));
      },
      handleError(error) {
        this.$notifier.showError(error);
      }
    }
  };
</script>
