<template>
  <span class="toggles">
    <label>Weigh words by:</label>
    <button v-for="option in weightOptions" value="option" v-bind:class="{'chosen': (weightChosen === option)}" v-on:click="$emit('weigh', option)" >{{option}}</button>
    <span class="refresh">
    	<button v-on:click="$emit('refresh')">&#8635;</button>
    	<label v-if="lastUpdated !== null" id="refreshLast">Last refreshed at {{lastUpdated}}</label>
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