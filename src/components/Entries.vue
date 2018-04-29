<template>
	<div class="entries" v-if="word != null">
    <div>
      Entries with {{word}}:
    </div>
    <div class="container">
      <div class="row aPost" v-for="entry in entries">
        <div class="upvotes col-1">
          &uarr; {{entry.upvotes}}
        </div>
        <div class="col-8">
          {{entry.subreddit}}: 
          <a :href="entry.link">{{entry.title}}</a>
          <span v-if="entry.nsfw" v-bind:class="{nsfwFlair: entry.nsfw}">NSFW</span><br /><br />
        </div>
        <div class="col-2 postOptions">
          <button id="visComs" v-on:click="$emit('visComs', entry)">Cloud replies</button>
          <div id="modOptions" v-if="isMod === true">
            <button id="modLock" v-if="entry.name.includes('t3')" v-on:click="$emit('modLock', entry)">Lock post</button><br />
            <button id="modRemove" v-on:click="$emit('modRemove', entry)">Remove entry</button>
          </div>
        </div>
      </div>
    </div>
	</div> 
</template>

<script>
import * as promise from "d3.promise";

export default {
  name: "Entries",
  data () {
    return {
      es: this.entries,
      mods: [[]]
    }
  },
  props:  [
    "word",
    "entries",
    "isMod"
  ]
}
</script>

<style lang="scss">
.entries	{
	margin-left: 2%;
  margin-right: 2%;
  margin-top: 1%;
}

.aPost  {
  margin-bottom: 1%;
}

.upvotes  {
  color: #FF8b60;
  font-weight: bold;
}

.nsfwFlair  {
  color: red;
}

.postOptions  {
  font-size: .7em;
}

.postOptions button  {
  border: 1px solid black;

}

#modOptions {
  margin-top: 5%;
}

#modRemove  {
  background-color: orangered;
}

#modLock  {
  background-color: orange;
}
</style>