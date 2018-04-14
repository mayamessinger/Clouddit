<template>
  <select id="timeO" v-bind:value="time" v-on:input="$emit('input', $event.target.value)">
		<option disabled value="">Time Range</option>
	  <option v-for="option in times" v-bind:value="option">
	    {{ option }}
	  </option>
	</select>
</template>

<script>
export default {
  name: "TimeO",
  props:  [
    "times",
    "time"
  ]
}
</script>

<style lang="scss">

</style>