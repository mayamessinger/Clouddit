<template>
    <select id="sort" v-bind:value="sort" v-on:input="$emit('input', $event.target.value)">
	  <option disabled value="">Sort</option>
	  <option v-for="option in sorts" v-bind:value="option">
	    {{ option }}
	  </option>
	</select>
</template>

<script>
export default {
  name: "Sort",
  props:  [
    "sorts",
    "sort"
  ]
}
</script>

<style lang="scss">

</style>