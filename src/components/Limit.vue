<template>
	<div>
		<label class="param"># of posts:</label>
		<input id="limit" type="range" min="1" max="500" step="1" v-bind:value="limit" v-on:input="$emit('input', $event.target.value)" />
		<limitNum :limit="limit" v-model="limit"></limitNum><br />
	</div>
</template>

<script>
import LimitNum from "./LimitNum.vue";

export default {
  name: 'Limit',
  props:  [
    "limit"
  ],
  components: {
  	LimitNum
  }
}
</script>

<style lang="scss">

</style>