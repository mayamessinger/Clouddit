<template>
  <div class="devInfo">
    <span>Developed by /u/D1sc0rd1a <a title="Reddit" href="https://reddit.com/user/D1sc0rd1a"><img src="./../assets/reddit_icon.png" alt="Reddit"/></a></span>
    <span>See this project on <a title="GitHub" href="https://github.com/mayamessinger/Clouddit"><img src="./../assets/github.svg" alt="GitHub"/></a></span>
    <div id="siteDesc">
      Clouddit is a data visualization site for Reddit. Word clouds created parse the words of posts (or comments) in a subreddit (with multireddit support) and display the cumulative data weighed by either number of upvotes a word has gotten (sum of upvotes for each post the word has appeared in) or number of occurrences (number of posts the word has appeared in).<br />Reddit users can sign in with OAuth2 and view their own front page and their own posts/comments. Moderators can sign in and visualize data in subreddits they moderate, and lock or remove threads. <br /> <b>Click on a word to view the posts/comments it appears in! The word cloud is also draggable and zoomable</b>
    </div>
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

#siteDesc {
  margin-left: 10%;
  margin-right: 10%;
  margin-top: 1%;
  width: 80%;
}
</style>