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
    <div class="source">
      <span id="clouding">Clouding children of: </span>{{title}}
    </div>
	</span>
</template>

<script>
export default {
  name: "GraphToggles",
  props:  [
    "lastUpdated",
  	"weightChosen",
    "weightOptions",
    "title"
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

.weightButtons  {
  float: left;
}

.chosen	{
	border: 2px solid #0077D6;
}

.source {
  display: block;
  font-family: Georgia;
  margin-left: auto;
  margin-right: auto;
  margin-top: 0.5%;
  text-align: center;
}

#clouding {
  color: blue;
  font-weight: bold;
}

.refresh	{
  float: right;
  margin-right: 1.5%;
}
</style>