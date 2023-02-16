<template>
  <component :is="gameid"></component>
</template>

<script>
import KenKen from './kenken/KenKen.vue';
import AlphaBeta from './alphabeta/AlphaBeta.vue';

export default {
  components: {
    KenKen, AlphaBeta
  },
  props: ['gameid'],
};

</script>
