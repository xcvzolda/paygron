<template>
  <div class="justify-center">

    <div class="w-9/12 text-center py-20 mb-5 -mt-24 border-2 border-dotted border-blue-400 rounded-md text-blue-400">
      <NFCReader v-model="playerId" />
      Bring player card here to<br>
      start a transaction
    </div>

    <Button class="border-red-800 text-red-800 fixed w-4/6" style="bottom: 30px" @click="endGame" variant="secondary" danger>
      End Game
    </Button>

  </div>
</template>

<script>
import NFCReader from '@/components/NFCReader.vue';
import Button from '@/components/Button.vue';

export default {
  name: 'Game',
  components: {
    NFCReader,
    Button,
  },
  data() {
    return {
      playerId: '',
    }
  },
  computed: {
    players() {
      return JSON.parse(localStorage.getItem('game'));
    }
  },
  watch: {
    playerId() {
      this.$router.push(`/transact/${this.playerId}`);
    }
  },
  methods: {
    endGame() {
      localStorage.removeItem('game');
      this.$router.replace('/');
    }
  }
}
</script>
