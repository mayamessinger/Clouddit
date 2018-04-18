<template>
  <span class="weights col-9">
    <label>Weigh words by:</label>
    <button v-for="option in weightOptions" value="option" v-bind:class="{'chosen': (weightChosen === option)}" v-on:click="$emit('weigh', option)" >{{option}}</button>
    <span class="refresh">
    	<button v-on:click="$emit('refresh'), newTime()">&#8635;</button>
    	<label v-if="refresh !== null" id="refreshLast">Last refreshed at {{refresh}}</label>
    </span>
	</span>
</template>

<script>
export default {
  name: "ChangeVisualize",
  data()	{
  	return {
  		refresh: null
  	}
  },
  props:  [
  	"weightChosen",
    "weightOptions"
  ],
  methods:	{
  	newTime()	{
  		var d = new Date();
  		var h = d.getHours();
  		var m = d.getMinutes();
  		var s = d.getSeconds();

  		if (h < 10)	{
  			h = "0" + h;
  		}
  		if (m < 10)	{
  			m = "0" + m;
  		}
  		if (s < 10)	{
  			s = "0" + s;
  		}

  		this.refresh = h + ":" + m + ":" + s;
  	}
  },
  mounted: function() {
  	this.newTime()
  }
}
</script>

<style lang="scss">
.weights	{
	font-size: 0.85em;
	height: 3%;
	width: 100%;
}

.weights button	{
}

.chosen	{
	background-color: gray;
}

.refresh	{

}
</style>