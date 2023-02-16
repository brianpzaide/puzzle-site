<template>
  <the-navigation :game-name="gameName" :home-isactive="homePage" @navigation-events="navigationEvents"></the-navigation>
  <main>
    <section v-if="homePage">
      <ul>
        <game-item
          v-for="game in games"
          :key="game.gameid"
          :gameid="game.gameid"
          :name="game.name"
          @gameselect-events="gameselectEvents"
        ></game-item>
      </ul>
    </section>

    <section v-else>
      <game-holder :gameid="gameId"></game-holder> 
    </section>

  </main>

</template>

<script>
import TheNavigation from './components/TheNavigation.vue';
import GameItem from './components/GameItem.vue';
import GameHolder from './components/GameHolder.vue';

export default {
  name: 'App',
  components: {
    TheNavigation, GameItem, GameHolder
  },
  data(){
    return {
      games: [
        { gameid: 'ken-ken', name: 'Ken Ken'},
        { gameid: 'alpha-beta', name: 'Alpha Beta'},
      ],
      homePage: true,
      gameName: "Game",
      gameId: ""
    };
  },
  provide() {
    return {
      games: this.games,
    };
  },
  methods:{
    navigationEvents(toHomePage){
      this.homePage = toHomePage
    },
    gameselectEvents(selectedGameName, selectedGameId){
      this.gameName = selectedGameName
      this.gameId = selectedGameId
      this.homePage = false
    }
  }
}
</script>

<style scoped>
* {
  box-sizing: border-box;
}

html {
  font-family: sans-serif;
}

body {
  margin: 0;
}

ul{
  list-style: none;
  margin: 2rem auto;
  max-width: 40rem;
  padding: 0;
}

</style>
