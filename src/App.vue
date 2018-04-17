<template>
  <div id="app" class="container">
    <div class="row">
      <myCloud class="col-9" id="cloud">
      </myCloud>
      <div class="params col-3">
        <subreddit :subreddit="subreddit" v-model="subreddit"></subreddit>
        <sort :sort="sort" :sorts="sorts" v-model="sort"></sort>
        <limit :limit="limit" v-model="limit"></limit>
        <timeO :time="time" :times="times" :sort="sort" v-model="time"></timeO>
        <input class="param" type="button" v-on:click="setQuery" value="Visualize" />
        <changeWeight :weightOptions="weightOptions" @weigh="weigh($event)">
        </changeWeight>
        <stopList :stopList="excluded" :oldStop="prevExcluded" @addBackWord="addBackWord($event)">
        </stopList>
      </div>
      <posts class="posts col-9" :word="wordSelected" :posts="postsSelected">
      </posts>
    </div>
  </div>
</template>

<script>
import excluded from "./assets/stopList2.json";
import $ from "jquery";
import * as d3 from "d3";
import * as cloud from "d3-cloud";
import * as promise from "d3.promise";

import MyCloud from "./components/Cloud.vue";
import Posts from "./components/Posts.vue";
import Subreddit from "./components/Subreddit.vue";
import Sort from "./components/Sort.vue";
import Limit from "./components/Limit.vue";
import TimeO from "./components/Time.vue";
import ChangeWeight from "./components/ChangeWeight.vue";
import StopList from "./components/StopList.vue";

var domParser = new DOMParser;
const redditDomain = "https://www.reddit.com/"

export default {
  name: 'app',
  data() {
    return {
      subUser: "/r/",
      subreddit: "all",
      sort: "hot",
      sorts: ["best", "hot", "new", "controversial", "top", "rising"],
      limit: 250,
      time: "",
      times: ["hour", "day", "week", "month", "year", "all"],
      redditUrl: redditDomain + this.subUser + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time,
      map: [],
      excluded: excluded.words,
      prevExcluded: [],
      wordSelected: null,
      postsSelected: [],
      weightOption: "upvotes",
      weightOptions: ["upvotes", "occurrences"]
    }
  },
  components: {
    MyCloud,
    Posts,
    Subreddit,
    Sort,
    Limit,
    TimeO,
    ChangeWeight,
    StopList
  },
  methods:  {
    // update query to reflect changed values
    setQuery() {
      this.redditUrl = redditDomain + this.subUser + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time;

      setTimeout(this.ready, 100);
    },
    // call make map
    ready() {
      this.map = [];
      this.wordSelected = null;
      this.postsSelected = [];
      this.getPostsData(this.redditUrl).then(this.makeCloud());
    },
    // get values from reddit JSON
    getPostsData(url)  {
      return promise.json(url, posts => {
              posts.data.children.forEach(post => {
                post.data.title.split(" ").forEach( word =>  {
                  word = this.prettify(word);

                  if (this.isExcluded(word)) {
                    return;
                  }
                  
                  this.addToMap(word, post);
                });
              });
            });
    },
    // check if word should be added to stored data, then do so if it hsould
    addToMap(word, post)  {
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
          if (loggedPost.link === redditDomain + post.data.permalink) {
            newPost = false;
          }
        });

        if (newPost) {
          this.map[i].upvotes += post.data.ups;
          this.map[index].posts.push({title: domParser.parseFromString(post.data.title, "text/html").body.textContent, 
                                      subreddit: post.data.subreddit_name_prefixed,
                                      link: redditDomain + post.data.permalink,
                                      upvotes: post.data.ups});
        }
      }
      else  {
        this.map.push({word: word,
                      occurrences: 1,
                      upvotes: post.data.ups,
                      posts:
                        [{title: domParser.parseFromString(post.data.title, "text/html").body.textContent,
                          subreddit: post.data.subreddit_name_prefixed,
                          link: redditDomain + post.data.permalink,
                          upvotes: post.data.ups}]});
      }
    },
    // universal formatting to make word cloud words not repeat
    prettify(word) {
      word = domParser.parseFromString(word, "text/html").body.textContent; // parse HTML entites (like &nbsp;)

      var specialChars = "!@#*()[]{}|:;<>?,.\"";
      for (var i = 0; i < specialChars.length; i++) {
        word = word.replace(new RegExp("\\" + specialChars[i], "gi"), "");
      }

      word = word.toUpperCase();

      return word;
    },
    // checks stopList, list of common words that will "clutter" a word cloud
    isExcluded(word) {
      if (this.excluded.length > 1) {
        if (this.excluded.includes(word)) {
          return true;
        }
        return false;
      }
      else  {
        setTimeout(this.isExcluded(word), 100);
      }
    },
    // allows word to be added to or removed from stop list
    addBackWord(word) {
      if (this.excluded.includes(word)) {
        this.excluded.splice(this.excluded.indexOf(word), 1);
        this.prevExcluded.push(this.prettify(word));
      }
      else if (this.prevExcluded.includes(word))  {
        this.excluded.splice(0, 0, this.prettify(word));
        this.prevExcluded.splice(this.prevExcluded.indexOf(word), 1);
      }
      else  {
        this.prevExcluded.splice(0, 0, this.prettify(word));
      }

      this.excluded.sort();
      this.prevExcluded.sort();

      setTimeout(this.ready(), 100);
    },
    // set up to visualize cloud from word map
    makeCloud() {
      d3.select("svg").remove();

      if (this.map.length > 1) {
        var words = this.decideWeight();

        cloud()
          .size([screen.width * 4/5, screen.height * 4/5])  // size of cloud
          .words(words) // words to use in cloud
          .rotate(0)  // rotation of words
          .fontSize(function(d) { return d.size; }) // operates on each word
          .on("end", this.drawCloud)  // after making cloud, make image
          .start(); // make cloud
      }
      else  {
        setTimeout(this.makeCloud, 100);
      }
    },
    // changew how to weigh words
    weigh(weight) {
      if (this.weightOption !== weight) {
        this.weightOption = weight;
        this.setQuery();
      }
    },
    // update word size mapping to reflect weight option
    decideWeight() {
      if (this.weightOption === "upvotes") {
        var words = this.map
          .map(function(d) {
              return {text: d.word,
                      occurrences: d.occurrences,
                      upvotes: d.upvotes,
                      posts: d.posts,
                      size: d.upvotes/1000};
          });
      }
      else if (this.weightOption === "occurrences")  {
        var words = this.map
        .map(function(d) {
            return {text: d.word,
                    occurrences: d.occurrences,
                    upvotes: d.upvotes,
                    posts: d.posts,
                    size: d.occurrences * 10};
        });
      }

      return words;
    },
    // make cloud SVG and display
    drawCloud(words) {
      var color = d3.scaleOrdinal(d3.schemeCategory10);

      // http://bl.ocks.org/Kcnarf/ccf9bef19eb85ed6a3a7f99824c2ccb4
      
      d3.select(".cloud")
          .append("svg")  // make HTML component
          .attr("width", $("#cloud").width())  // size of container (svg)
          .attr("height", $("#cloud").height())  // size of container (svg)
          .attr("class", "wordcloud") // maake HTML class
          .append("g")  // container element for svg
          .attr("transform", "translate(" + $("#cloud").width() / 2 + "," + $("#cloud").height() / 2 + ")") // set center point of cloud
          .attr("text-anchor", "middle")
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
          .on("click", d => {
            this.wordSelected = d.text;
            this.displayPosts(d.posts);
          });
    },
    end(words) { 
      console.log(JSON.stringify(words));
    },
    // when word is clicked, show posts with word in them
    displayPosts(posts) {
      this.postsSelected = [];
      posts.forEach(post => {
        this.postsSelected.push(post);
      });
    }
  },
  mounted: function() {
    this.setQuery();
  }
}
</script>

<style lang="scss">
html  {
  height: 100%;
  width: 100%;
}

body  {
  height: 100%;
  width: 100%;
}

#app  {
  height: 100%;
  padding: 0;
  width: 100%;
}

.container  {
  margin: 0 0 0 0;
  max-width: 100%;
}

.row  {
  height: 100%;
  width: 100%;
}

.params {
  margin-top: 3%;
}

.params input {
  border-color: black;
  border-left: none;
  border-right: none;
  border-top: none;
}

.params select {
  border: none;
}

.param   {
  margin-left: 1%;
}
</style>
