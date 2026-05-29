<template>
  <div class="justify-start">
    <h3 class="text-green-500 font-semibold">
      {{ formattedAmount }}
    </h3>

    <form class="mt-32 w-9/12" @submit.prevent="transact">
      <div class="flex">
        <MegaChoice
          name="type"
          inputValue="pay"
          v-model="type"
          overrideClass="border-red-500 text-red-500"
        >
          Pay
        </MegaChoice>
      </div>

      <h3 class="text-green-500 font-semibold">
        {{ new Intl.NumberFormat('en-IN').format(currentPlayer?.amount || 0) }}
      </h3>

      <FormInput class="my-10" v-model="amount" type="number" required>
        Amount
      </FormInput>

      <div class="text-gray-500">
        <span v-if="type === 'pay'">To</span>
      </div>

      <div class="flex">
        <MegaChoice
          v-for="(player, id) in opponents"
          :key="id"
          name="to"
          :inputValue="id"
          v-model="to"
          :disabled="type === 'collect'"
        >
          {{ player.name }}
        </MegaChoice>
      </div>

      <Button
        type="submit"
        class="fixed w-4/6 transform -translate-x-1/2"
        style="bottom: 30px; left: 50%"
        :class="type === 'pay' ? 'bg-red-500' : 'bg-green-500'"
      >
        <span v-if="type === 'pay'">Pay</span>
        <span v-if="type === 'collect'">Collect</span>
      </Button>
    </form>
  </div>
</template>

<script>
import FormInput from '@/components/FormInput.vue';
import MegaChoice from '@/components/MegaChoice.vue';
import Button from '@/components/Button.vue';

export default {
  name: 'Transact',

  components: {
    FormInput,
    MegaChoice,
    Button,
  },

  data() {
    return {
      type: 'pay',
      to: '0', // "0" = bank / disabled target
      amount: 0,
    };
  },

  computed: {
    activePlayerId() {
      return this.$route.params.id || '0';
    },

    players() {
      try {
        return JSON.parse(localStorage.getItem('game') || '{}');
      } catch (e) {
        return {};
      }
    },

    currentPlayer() {
      return this.players?.[this.activePlayerId];
    },

    opponents() {
      const { [this.activePlayerId]: _, ...rest } = this.players;
      return rest;
    },

    formattedAmount() {
      const amount = this.currentPlayer?.amount || 0;
      return new Intl.NumberFormat('en-IN').format(amount);
    },
  },

  methods: {
    transact() {
      const players = { ...this.players };

      const activePlayer = players[this.activePlayerId];
      if (!activePlayer) return;

      const amount = Number(this.amount);
      if (isNaN(amount) || amount <= 0) return;

      activePlayer.transactions = activePlayer.transactions || [];

      let finalAmount = this.type === 'pay' ? -amount : amount;

      activePlayer.amount =
        (activePlayer.amount || 0) + finalAmount;

      activePlayer.transactions.push({
        type: this.type,
        amount: finalAmount,
      });

      // "0" = bank, so ignore it
      if (this.to && this.to !== '0' && players[this.to]) {
        players[this.to].transactions =
          players[this.to].transactions || [];

        players[this.to].amount =
          (players[this.to].amount || 0) - finalAmount;

        players[this.to].transactions.push({
          type: this.type === 'pay' ? 'collect' : 'pay',
          amount: finalAmount,
        });
      }

      localStorage.setItem('game', JSON.stringify(players));
      this.$router.replace('/game');
    },
  },
};
</script>
