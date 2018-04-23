<template>
  <div class="redditLogin">
    <button class="btn btn-secondary" id="loginButton" @click="$emit('login', $event.target.value)">Login with Reddit</button>
  </div>
</template>

<script>
export default {
  name: "Login",
  props:  [
  ]
}
</script>

<style lang="scss">
.redditLogin  {
  display: block;
  font-size: 0.85em;
  height: 3%;
  width: 100%;
}

#loginButton  {
  float: right;
}
</style>