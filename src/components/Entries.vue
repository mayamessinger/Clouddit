<template>
	<div class="entries" v-if="word != null">
    <div>
      Entries with {{word}}:
    </div>
    <div class="container">
      <div class="row" v-for="entry in entries">
        <div class="upvotes col-1">
          &uarr; {{entry.upvotes}}
        </div>
        <div class="col-8">
          {{entry.subreddit}}: 
          <a :href="entry.link">{{entry.title}}</a>
          <span v-if="entry.nsfw" v-bind:class="{nsfwFlair: entry.nsfw}">NSFW</span><br /><br />
        </div>
        <div class="col-2">
          <button id="visComs" v-on:click="$emit('visComs', entry)">Cloud replies</button>
        </div>
      </div>
    </div>
	</div>
</template>

<script>
export default {
  name: "Entries",
  data () {
    return {
    }
  },
  props:  [
    "word",
    "entries"
  ]
}
</script>

<style lang="scss">
.entries	{
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

#visComs  {
  font-size: .7em;
}
</style>