<template>
  <span class="toggles">
    Weigh words by:
    <span id="weightButtons">
      <button v-for="option in weightOptions" value="option" v-bind:class="{'chosen': (weightChosen === option)}" v-on:click="$emit('weigh', option)" >{{option}}</button>
    </span>
    <span class="refresh">
    	<button id="refreshButton" v-on:click="$emit('refresh')">&#8635;</button>
    	<span id="refreshLast" v-if="lastUpdated !== null">Last refreshed at {{lastUpdated}}</span>
    </span>
	</span>
</template>

<script>
export default {
  name: "GraphToggles",
  props:  [
    "lastUpdated",
  	"weightChosen",
    "weightOptions"
  ]
}
</script>

<style lang="scss">
.toggles	{
  display: block;
	font-size: 0.85em;
	height: 3%;
  padding: 0% 1.5%;
	width: 100%;
}

button  {
}

.chosen	{
	border: 2px solid #0077D6;
}

.refresh	{
  float: right;
  margin-right: 1.5%;
}
</style>