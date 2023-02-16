<template>
  <component :is="gameid"></component>
</template>

<script>
import KenKen from './kenken/KenKen.vue';
import ExampleExample from './exampleexample/ExampleExample.vue';

export default {
  components: {
    KenKen, ExampleExample
  },
  props: ['gameid'],
};

</script>
