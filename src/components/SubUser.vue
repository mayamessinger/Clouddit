<template>
  <span class="param">
    <select id="sr" v-bind:value="sr" v-on:input="$emit('input', $event.target.value)">
      <option>
        r/
      </option>
      <option>
        user/
      </option>
    </select>
	</span>
</template>

<script>
export default {
  name: "SubUser",
  props:  [
    "sr"
  ]
}
</script>

<style lang="scss">
</style>