<template>
  <div class="devInfo">
    <span>Developed by /u/D1sc0rd1a <a title="Reddit" href="https://reddit.com/user/D1sc0rd1a"><img src="./../assets/reddit_icon.png" alt="Reddit"/></a></span>
    <span>See this project on <a title="GitHub" href="https://github.com/mayamessinger"><img src="./../assets/github.svg" alt="GitHub"/></a></span>
  </div>
</template>

<script>
export default {
  name: "DevInfo",
  props:  [
  ]
}
</script>

<style lang="scss">
.devInfo  {
  text-align: center;
  margin-bottom: 2%;
}

.devInfo span  {
  margin: 0 2%;
}

.devInfo img {
  max-height: 1%;
  max-width: 1%;
}
</style>