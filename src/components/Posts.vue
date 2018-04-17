<template>
	<div class="posts" v-if="word != null">
    <div>
      Posts with {{word}}:
    </div>
    <div class="container">
      <div class="row" v-for="post in posts">
        <div class="col-1">
          Up
        </div>
        <div class="col-7">
          {{post.subreddit}}: 
          <a :href="post.link">{{post.title}}</a><br /><br />
        </div>
      </div>
    </div>
	</div>
</template>

<script>
export default {
  name: 'Posts',
  data () {
    return {
    }
  },
  props:  [
    "word",
    "posts"
  ]
}
</script>

<style lang="scss">
.posts	{
	margin-left: 2%;
  margin-right: 2%;
  margin-top: 1%;
}
</style>