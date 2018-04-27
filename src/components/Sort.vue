<template>
  <div v-if="ru === 'r/'">
    <label class="param" for="sort">Sort:</label>
    <select id="sort" v-bind:value="sort" v-on:input="$emit('input', $event.target.value)">
      <option disabled value="">Sort</option>
      <option v-for="option in sorts" v-bind:value="option">
        {{ option }}
      </option>
    </select>
  </div>
</template>

<script>
export default {
  name: "Sort",
  props:  [
    "sorts",
    "sort",
    "ru"
  ]
}
</script>

<style lang="scss">

</style>