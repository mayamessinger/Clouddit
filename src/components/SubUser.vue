<template>
  <div class="param">
    <label for="ru">Type:</label>
    <select id="ru" v-bind:value="ru" v-on:input="$emit('input', $event.target.value)">
      <option>
        r/
      </option>
      <option>
        user/
      </option>
    </select>
	</div>
</template>

<script>
export default {
  name: "SubUser",
  props:  [
    "ru"
  ]
}
</script>

<style lang="scss">
</style>