<template>
  <div v-if="sort === 'top' || sort === 'controversial'">
    <label class="param">Time range:</label>
    <select id="timeO" v-bind:value="time" v-on:input="$emit('input', $event.target.value)">
  		<option disabled value="">Time Range</option>
  	  <option v-for="option in times" v-bind:value="option">
  	    {{ option }}
  	  </option>
  	</select>
</div>
</template>

<script>
export default {
  name: "TimeO",
  props:  [
    "times",
    "time",
    "sort"
  ]
}
</script>

<style lang="scss">

</style>