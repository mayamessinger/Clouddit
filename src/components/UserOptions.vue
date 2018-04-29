<template>
  <div class="redditUser dropdown">
    <button class="btn btn-secondary dropdown-toggle" id="userDropdown" type="button" data-toggle="dropdown" aria-haspopup="true" aria-expanded="false">
      <img src="./../assets/snoo.png" alt="snoo" height="15">
      &nbsp;{{username}}
    </button>
    <ul class="dropdown-menu pull-right" aria-labelledby="userDropdown">
      <li class="dropdown-item"><a href="#" @click="$emit('frontPage')">Cloud my front page</a></li>
      <li class="dropdown-item"><a href="#" @click="$emit('userPosts')">Cloud my posts</a></li>
      <li class="dropdown-item"><a href="#" @click="$emit('userComments')">Cloud my comments</a></li>
      <li class="dropdown-item"><a href="#" @click="$emit('logout')">Logout</a></li>
    </ul>
  </div>
</template>

<script>
import snooIcon from "../assets/snoo.png";

export default {
  name: "UserOptions",
  props:  [
    "username"
  ]
}
</script>

<style lang="scss">
.redditUser  {
  display: block;
  font-size: 0.85em;
  height: 3%;
  position: relative;
  width: 100%;
}

#userDropdown  {
  float: right;
}
</style>