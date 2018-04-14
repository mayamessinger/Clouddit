<template>
  <div id="app" class="container">
    <input type="button" v-on:click="setQuery"/>
    <myCloud
      :subreddit="subreddit"
      :sort="sort"
      :limit="limit"
      :time="time">
    </myCloud>
  </div>
</template>

<script>
import excluded from "./assets/stopList2.json";
import * as d3 from "d3";
import * as cloud from "d3-cloud";
import MyCloud from "./components/Cloud.vue";

export default {
  name: 'app',
  data () {
    return {
      subreddit: "all",
      sort: "hot",
      limit: 300,
      time: "all",
      redditUrl: "https://www.reddit.com/r/" + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time,
      map: [{word: "blah", occurrences: 5, posts:{}}]
    }
  },
  components: {
    MyCloud
  },
  methods:  {
    setQuery() {
      // this.map = [];
      // this.subreddit = document.getElementById("subreddit").value;
      // this.sort = document.getElementById("sort").value;
      // this.limit = document.getElementById("limit").value;
      // this.time = document.getElementById("time").value;

      d3.select("svg").remove();

      this.redditUrl = "https://www.reddit.com/r/" + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time;
      this.getPostsData(this.redditUrl);
      this.makeCloud();
      // makeBar();
    },
    getPostsData(url)  {
      d3.json(url, posts => {
                  posts.data.children.forEach(post => {
                    post.data.title.split(" ").forEach( word =>  {
                      word = prettify(word);

                      console.log(word);

                      if (isExcluded(word)) {
                        return;
                      }

                      var index = null;
                      for (var i = 0; i < this.map.length; i++)  {
                        if (this.map[i].word === word) {
                          index = i;
                          break;
                        }
                      }
                      
                      if (index !== null) {
                        this.map[i].occurrences++;

                        var newPost = true;
                        this.map[i].posts.forEach(loggedPost => {
                          if (loggedPost.title === post.data.title && loggedPost.subreddit === post.data.subreddit_name_prefixed) {
                            newPost = false;
                          }
                        });

                        if (newPost) {
                          this.map[index].posts.push({title: post.data.title, subreddit: post.data.subreddit_name_prefixed, link: "https://reddit.com" + post.data.permalink});
                        }
                      }
                      else  {
                        this.map.push({word: word, occurrences: 1, posts: [{title: post.data.title, subreddit: post.data.subreddit_name_prefixed, link: "https://www.reddit.com" + post.data.permalink}]});
                      }
                    });
                  });
                });
      console.log(this.map);
    },
    prettify(word) {
      var specialChars = "!@#*()[]{}|:;<>?,.\"";
      word = word.replace(/&amp;/g, "&");

      for (var i = 0; i < specialChars.length; i++) {
        word = word.replace(new RegExp("\\" + specialChars[i], "gi"), "");
      }

      word = word.toUpperCase();

      return word;
    },
    isExcluded(word) {
      if (this.excluded.length > 1) {
        if (this.excluded.includes(word)) {
          return true;
        }
        return false;
      }
      else  {
        setTimeout(this.isExcluded(word), 500);
      }
    },
    makeCloud() {
      // if (this.map.length > 1) {
        var words = this.map
          .map(function(d) {
              return {text: d.word, posts: d.posts, size: d.occurrences * 10};
          });

        cloud()
                .size([screen.width * 4/5, screen.height * 4/5])  // size of cloud
                .words(words) // words to use in cloud
                .rotate(0)  // rotation of words
                .fontSize(function(d) { return d.size; }) // operates on each word
                .on("end", this.draw)  // after making cloud, make image
                .start(); // make cloud
      // }
      // else  {
      //   setTimeout(this.makeCloud, 500);
      // }
    },
    draw(words) {
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      
      d3.select("body")
              .append("svg")  // make HTML component
              .attr("width", screen.width)  // size of container (svg)
              .attr("height", screen.height)  // size of container (svg)
              .attr("class", "wordcloud") // maake HTML class
              .append("g")  // container element for svg
              .attr("transform", "translate(" + screen.width / 2 + "," + screen.height / 2 + ")") // set center point of cloud
              .selectAll("text")  // select all child elements
              .data(words)  // data to use
              .enter().append("text") // get data missing elements
              .style("font-size", function(d) { return d.size + "px"; })  // use sizes set before
              .style("fill", function(d, i) { return color(i); }) // get colorscheme
              .attr("transform", function(d) {
                  return "translate(" + [d.x, d.y] + ")rotate(" + d.rotate + ")"; // rotate words individually if desired
              })
              .text(d => { return d.text; })  // set text content
              .on('mouseover', function(d){
                  var nodeSelection = d3.select(this).style("font-size", function(d) { return d.size + 20 + "px"; });
              })
              .on('mouseout', function(d){
                  var nodeSelection = d3.select(this).style("font-size", function(d) { return d.size + "px"; });
              })
              .on("click", d => {displayPosts(d.posts);});
    },
    end(words) { 
      console.log(JSON.stringify(words));
    },
    displayPosts(posts) {
      var jankLinks = "";

      posts.forEach(post => {
        jankLinks += "<a href='" + post.link + "'>" + post.title + "</a><br /><br />";
      });
    var myWindow = window.open("", "Posts with this word", "width=500,height=300");
    myWindow.document.write(jankLinks);
    }
  },
  mounted: this.setQuery
}
</script>

<style lang="scss">

</style>
