<template>
  <UiModal :open="open" @close="$emit('close')" class="d-flex">
    <form @submit.prevent="handleSubmit" class="d-flex flex-column flex-auto">
      <h3 class="m-4 mb-0 text-center">Confirm vote</h3>
      <h4 class="m-4 mb-0 text-center">
        Are you sure you want to vote "{{
          proposal.msg.payload.choices[selectedChoice - 1]
        }}"? <br />This action <b>cannot</b> be undone.
      </h4>
      <div class="m-4 p-4 border rounded-2 text-white">
        <div class="d-flex">
          <span v-text="'Option'" class="flex-auto text-gray mr-1" />
          {{ proposal.msg.payload.choices[selectedChoice - 1] }}
        </div>
        <div class="d-flex">
          <span v-text="'Snapshot'" class="flex-auto text-gray mr-1" />
          <a
            :href="_etherscanLink(proposal.msg.payload.snapshot, 'block')"
            target="_blank"
            class="float-right"
          >
            {{ $n(proposal.msg.payload.snapshot) }}
            <Icon name="external-link" class="ml-1" />
          </a>
        </div>
        <div class="d-flex">
          <span
            v-text="'Total voting power'"
            class="flex-auto text-gray mr-1"
          />
          <span
            v-text="`${_numeral(gov.votingPower)} ${gov.namespace.symbol}`"
          />
        </div>
      </div>
      <div class="p-4 overflow-hidden text-center border-top">
        <div class="col-6 float-left pr-2">
          <UiButton @click="$emit('close')" type="button" class="width-full">
            Cancel
          </UiButton>
        </div>
        <div class="col-6 float-left pl-2">
          <UiButton
            :disabled="loading"
            :loading="loading"
            type="submit"
            class="width-full button--submit"
          >
            Vote
          </UiButton>
        </div>
      </div>
    </form>
  </UiModal>
</template>

<script>
import { mapActions } from 'vuex';
import namespaces from '@/namespaces.json';

export default {
  props: ['open', 'namespace', 'proposal', 'id', 'selectedChoice', 'snapshot'],
  data() {
    return {
      loading: false,
      namespaces
    };
  },
  computed: {
    symbol() {
      return this.namespace.symbol || this._shorten(this.namespace.token);
    }
  },
  methods: {
    ...mapActions(['send']),
    async handleSubmit() {
      this.loading = true;
      await this.send({
        token: this.namespace.token,
        type: 'vote',
        payload: {
          proposal: this.id,
          choice: this.selectedChoice
        }
      });
      this.$emit('reload');
      this.$emit('close');
      this.loading = false;
    }
  }
};
</script>
