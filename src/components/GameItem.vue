<template>
  <li>
    <h3 @click="goPlay">{{ name }}</h3>
  </li>
</template>

<script>
export default {
  props: ['gameid', 'name'],
  methods:{
    goPlay(){
        this.$emit("gameselect-events", this.name, this.gameid)
    }
  }
};
</script>

<style scoped>
li {
  margin: 1rem 0;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.26);
  border-radius: 12px;
  padding: 1rem;
}

li h3 {
  text-align: center;
  margin: 0;
  font-size: 1.25rem;
  color: #00a876;
}

li h3:hover,
li h3:active  {
  color: #f1a80a;
  cursor: pointer;
}
</style>