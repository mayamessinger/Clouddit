<template>
	<div class="posts" v-if="word != null">
    <div>
      Posts with {{word}}:
    </div>
    <div class="container">
      <div class="row" v-for="post in posts">
        <div class="upvotes col-1">
          &uarr; {{post.upvotes}}
        </div>
        <div class="col-8">
          {{post.subreddit}}: 
          <a :href="post.link">{{post.title}}</a>
          <span v-if="post.nsfw" v-bind:class="{nsfwFlair: post.nsfw}">NSFW</span><br /><br />
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

.upvotes  {
  color: #FF8b60;
  font-weight: bold;
}

.nsfwFlair  {
  color: red;
}
</style>