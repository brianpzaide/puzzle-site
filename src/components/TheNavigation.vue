<template>
  <header>
    <nav>
      <ul>
        <li>
            <h3 @click="homeClicked" :class="{ active: homeIsactive }">Home</h3>
        </li>
        <li>
          <h3 @click="currentGameClicked" :class="{ active: !homeIsactive }">{{gameName}}</h3>
        </li>
      </ul>
    </nav>
  </header>
</template>
<script>
    export default{
        props: [
            'gameName', 'homeIsactive'
        ],
        methods: {
            homeClicked(){
                this.$emit("navigation-events", true)
            },
            currentGameClicked(){
                this.$emit("navigation-events", false)
            }
        }
    }
</script>


<style scoped>
header {
  width: 100%;
  height: 5rem;
  background-color: #901a1a;
}

nav {
  height: 100%;
}

ul {
  list-style: none;
  margin: 0;
  padding: 0;
  height: 100%;
  display: flex;
  justify-content: center;
  align-items: center;
}

li {
  margin: 0 2rem;
}

h3 {
  text-decoration: none;
  background: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  color: white;
  padding: 0.5rem 1.5rem;
  display: inline-block;
}

h3:hover,
h3:active{
  color: #f1a80a;
  border-color: #f1a80a;
  background-color: #00a876;
}

.active{
  text-decoration: none;
  background: transparent;
  border: 1px solid transparent;
  cursor: pointer;
  padding: 0.5rem 1.5rem;
  display: inline-block;
  color: #f1a80a;
  border-color: #f1a80a;
  background-color: #00a876;
}
</style>