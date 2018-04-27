<template>
  <div id="app" class="container">
    <div id="toggleButtons" class="row">
      <graphToggles class="col-9" :lastUpdated="lastUpdated" :weightChosen="weightOption" :weightOptions="weightOptions" @weigh="weigh($event)" @refresh="ready()">
      </graphToggles>
      <login class="col-3" v-if="!loggedIn" @login="login()"></login>
      <userOptions class="col-3" v-if="loggedIn" :username="username" @frontPage="frontPage()" @userPosts="userPosts()" @userComments="userComments()" @logout="logout()"></userOptions>
    </div>
    <div class="row" id="visuals">
      <myCloud class="col-9" id="cloud">
      </myCloud>
      <div class="params col-3">
        <subUser :sr="subUser" v-model="subUser"></subUser>
        <subreddit :subreddit="subreddit" v-model="subreddit"></subreddit>
        <sort :sort="sort" :sorts="sorts" v-model="sort"></sort>
        <limit :limit="limit" v-model="limit"></limit>
        <timeO :time="time" :times="times" :sort="sort" v-model="time"></timeO>
        <input class="param" type="button" v-on:click="setQuery" value="Visualize" />
        <stopList :stopList="excluded" :oldStop="prevExcluded" @addBackWord="addBackWord($event)">
        </stopList>
      </div>
    </div>
    <div class="row">
      <posts class="posts col-9" :word="wordSelected" :posts="postsSelected">
      </posts>
    </div>
  </div>
</template>

<script>
import excluded from "./assets/stopList2.json";
import $ from "jquery";
import bootstrap from "bootstrap";
import * as firebase from "firebase";
import * as d3 from "d3";
import * as cloud from "d3-cloud";
import * as promise from "d3.promise";

import GraphToggles from "./components/GraphToggles.vue";
import Login from "./components/Login.vue";
import UserOptions from "./components/UserOptions.vue";
import MyCloud from "./components/Cloud.vue";
import Posts from "./components/Posts.vue";
import SubUser from "./components/SubUser.vue";
import Subreddit from "./components/Subreddit.vue";
import Sort from "./components/Sort.vue";
import Limit from "./components/Limit.vue";
import TimeO from "./components/Time.vue";
import StopList from "./components/StopList.vue";

var config = {
  apiKey: "AIzaSyB4jOZ7jdVqB7L71XhoSfHTQe7-imfGcfE",
  authDomain: "clouddit-479.firebaseapp.com",
  databaseURL: "https://clouddit-479.firebaseio.com",
  projectId: "clouddit-479",
  storageBucket: "clouddit-479.appspot.com",
  messagingSenderId: "746195801558"
};

var db = firebase.initializeApp(config).database();
var storageRef = firebase.storage().ref();
var dbCodesRef = db.ref("codes");

var domParser = new DOMParser;
const redditDomain = "https://www.reddit.com/";
const client_id = "GeDalotx_uwLig";
const client_secret = "0WCPPtvbnnqhYKUIDHfN4Wdns7M";
// const redirect_uri = "https://duke-compsci290-spring2018.github.io/final-project-team-44/";
const redirect_uri = "http://localhost:8080";

export default {
  name: "app",
  data() {
    return {
      loggedIn: false,
      username: null,
      userToken: null,
      subUser: "r/",
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
      highestOccurrences: 0,
      highestUpvotes: 0,
      lastUpdated: this.newTime(),
      weightOption: "upvotes",
      weightOptions: ["upvotes", "occurrences"]
    }
  },
  components: {
    Login,
    UserOptions,
    GraphToggles,
    MyCloud,
    Posts,
    SubUser,
    Subreddit,
    Sort,
    Limit,
    TimeO,
    StopList
  },
  methods:  {
    // update query to reflect changed values
    setQuery() {
      if (this.subUser === "user/") {
        this.redditUrl = redditDomain + this.subUser + this.subreddit + "/.json?limit=" + this.limit;
      }
      else if (this.subUser = "r/")  {
        this.redditUrl = redditDomain + this.subUser + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time;
      }

      console.log(this.redditUrl);

      setTimeout(this.ready, 100);
    },
    // call make map
    ready() {
      this.newTime();
      this.map = [];
      this.wordSelected = null;
      this.postsSelected = [];
      this.highestOccurrences = 0;
      this.highestUpvotes = 0;
      this.getPostsData(this.redditUrl).then(this.makeCloud());
    },
    // get last time query qas run for display
    newTime() {
      var d = new Date();
      var h = d.getHours();
      var m = d.getMinutes();
      var s = d.getSeconds();

      if (h < 10) {
        h = "0" + h;
      }
      if (m < 10) {
        m = "0" + m;
      }
      if (s < 10) {
        s = "0" + s;
      }

      this.lastUpdated = h + ":" + m + ":" + s;
    },
    // get values from reddit JSON
    getPostsData(url)  {
      return promise.json(url, posts => {
        if (posts === null) {
          this.emptySub();
          return;
        }

        this.parsePosts(posts);
      });
    },
    parsePosts(posts)  {
      posts.data.children.forEach(post => {
        // post
        if (post.kind === "t3") {
          post.data.title.split(" ").forEach( word =>  {
            word = this.prettify(word);

            if (this.isExcluded(word)) {
              return;
            }
            
            this.addToMap(word, post);
          });
        }
        // comment
        else if (post.kind === "t1")  {
          post.data.body.split(" ").forEach( word =>  {
            word = this.prettify(word);

            if (this.isExcluded(word)) {
              return;
            }
            
            this.addToMap(word, post);
          });
        }
      });
    },
    // alert about bad subreddit call
    emptySub()  {
      alert("Oh no! Looks like /r/" + this.subreddit + " doesn't exist!");
    },
    // check if word should be added to stored data, then do so if it should
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
        if (this.map[i].occurrences > this.highestOccurrences) {
          this.highestOccurrences = this.map[i].occurrences;
        }

        var newPost = true;
        this.map[i].posts.forEach(loggedPost => {
          if (loggedPost.link === redditDomain + post.data.permalink) {
            newPost = false;
          }
        });

        if (newPost) {
          this.map[i].upvotes += post.data.ups;
          if (this.map[i].upvotes > this.highestUpvotes) {
            this.highestUpvotes = this.map[i].upvotes;
          }

          if (post.data.title !== undefined) {
            this.map[index].posts.push({title: domParser.parseFromString(post.data.title, "text/html").body.textContent, 
                                        subreddit: post.data.subreddit_name_prefixed,
                                        link: redditDomain + post.data.permalink,
                                        upvotes: post.data.ups,
                                        nsfw: post.data.over_18});
          }
          else if (post.data.body !== undefined)  {
            this.map[index].posts.push({title: domParser.parseFromString(post.data.body, "text/html").body.textContent, 
                                        link: redditDomain + post.data.permalink,
                                        upvotes: post.data.ups});
          }
        }
      }
      else  {
        if (post.data.ups > this.highestUpvotes) {
            this.highestUpvotes = post.data.ups;
          }

        if (post.data.title !== undefined) {
          this.map.push({word: word,
                        occurrences: 1,
                        upvotes: post.data.ups,
                        posts:
                          [{title: domParser.parseFromString(post.data.title, "text/html").body.textContent,
                            subreddit: post.data.subreddit_name_prefixed,
                            link: redditDomain + post.data.permalink,
                            upvotes: post.data.ups,
                            nsfw: post.data.over_18}]});
        }
        else if (post.data.body !== undefined)  {
          this.map.push({word: word,
                        occurrences: 1,
                        upvotes: post.data.ups,
                        posts:
                          [{title: domParser.parseFromString(post.data.body, "text/html").body.textContent,
                            link: redditDomain + post.data.permalink,
                            upvotes: post.data.ups}]});
        }
      }
    },
    // universal formatting to make word cloud words not repeat
    prettify(word) {
      word = domParser.parseFromString(word, "text/html").body.textContent; // parse HTML entites (like &nbsp;)

      var specialChars = "!@#*()[]{}|:;<>?,.\"`";
      for (var i = 0; i < specialChars.length; i++) {
        word = word.replace(new RegExp("\\" + specialChars[i], "gi"), "");
      }

      word = word.toUpperCase();

      return word;
    },
    // checks stopList, list of common words that will "clutter" a word cloud
    isExcluded(word) {
      if (this.excluded.length > 0) {
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
        this.excluded.push(this.prettify(word));
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

      if (this.map.length > 0) {
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
          .map(d => {
            return {text: d.word,
                    occurrences: d.occurrences,
                    upvotes: d.upvotes,
                    posts: d.posts,
                    size: d.upvotes / this.highestUpvotes * 100};
          });
      }
      else if (this.weightOption === "occurrences")  {
        var words = this.map
        .map(d => {
          return {text: d.word,
                  occurrences: d.occurrences,
                  upvotes: d.upvotes,
                  posts: d.posts,
                  size: d.occurrences / this.highestOccurrences * 100};
        });
      }

      return words;
    },
    // calculate font sizes - relation to max instead of hard-coded
    calcTextSize(value)  {
      if (this.weightOption === "upvotes") {
        return (value/this.highestUpvotes) * 100;
      }
      else if (this.weightOption === "occurrences")  {
        return (value/this.highestOccurrences) * 40;
      }
    },
    // make cloud SVG and display
    drawCloud(words) {
      var color = d3.scaleOrdinal(d3.schemeCategory10);
      
      var svg = d3.select(".cloud")
          .append("svg")  // make HTML component
          .attr("width", $("#cloud").width())  // size of container (svg)
          .attr("height", $("#cloud").height())  // size of container (svg)
          .attr("class", "wordcloud"); // maake HTML class
      
      var g = svg.append("g");

      var zoom = d3.zoom()
          .scaleExtent([0.73, 5])
          .on("zoom", function() {
            g.attr("transform", d3.event.transform);
          });

      svg.call(zoom);

      g
        .append("g")  // container element for svg
        .attr("transform", "translate(" + $("#cloud").width() / 2 + "," + $("#cloud").height() / 3 + ")") // set center point of cloud
        .attr("text-anchor", "middle")
        .selectAll("text")  // select all child elements
        .data(words)  // data to use
        .enter().append("text") // get data missing elements
        .style("font-size", d => { return d.size + "px"; })  // use sizes set before
        .style("fill", function(d, i) { return color(i); }) // get colorscheme
        .attr("transform", d => {
            return "translate(" + [d.x, d.y] + ")rotate(" + d.rotate + ")"; // rotate words individually if desired
        })
        .text(d => { return d.text; })  // set text content
        .on("mouseover", function(d){
            var nodeSelection = d3.select(this).style("font-size", function(d) { return d.size + 20 + "px"; });

        })
        .on("mouseout", function(d){
            var nodeSelection = d3.select(this).style("font-size", function(d) { return d.size + "px"; });
        })
        .on("click", d => {
          this.wordSelected = d.text;
          this.displayPosts(d.posts);
        });
    },
    // when word is clicked, show posts with word in them
    displayPosts(posts) {
      this.postsSelected = [];
      posts.forEach(post => {
        this.postsSelected.push(post);
      });
    },
    // login with reddit OAuth2
    login() {
      var response_type = "code"; // what want back from reddit
      var state = Math.random().toString(36).substring(2);  // random string, check received back against sent to verify same request
      var duration = "temporary"; // 1 hour
      var scope = "identity mysubreddits modposts";  // what this app wants to access

      window.open("https://www.reddit.com/api/v1/authorize?client_id=" + client_id +
                  "&response_type=" + response_type +
                  "&state=" + state +
                  "&redirect_uri=" + redirect_uri +
                  "&duration=" + duration +
                  "&scope=" + scope, "_self");
    },
    // check if user has logged in with reddit, to start opening up features to them
    checkLogin()  {
      var query = window.location.href;
      if (query.includes("state")) {  // if loaded from reddit after oauth
        var params = query.split("/?");
        var args = params[1].split("&");  // array of params, after domain

        var errorEXP = /error=([a-z_]*)/i;
        var error = null;
        var stateEXP = /state=(\S*)/i;
        var state = null;
        var codeEXP = /code=(\S*)/i;
        var code = null;

        args.forEach(param => {
          if (param.match(errorEXP) != null) {
            error = param.match(errorEXP)[1];
          }
          else if (param.match(stateEXP) != null) {
            state = param.match(stateEXP)[1];
          }
          else if (param.match(codeEXP) != null) {
            code = param.match(codeEXP)[1];
          }
        });

        if (error != null) {
          console.log(error + " for " + state);
          if (error === "access_denied") {
            alert("Login request was not approved");
          }
          else if (error === "unsupported_response_type")  {
            alert("Unable to log in, but it's the developer's fault, not yours.");
            console.log("response_type not token");
          }
          else if (error === "invalid_scope")  {
            alert("Unable to log in, but it's the developer's fault, not yours.");
            console.log("invalid_scope");
          }
          else if (error === "invalid_request")  {
            alert("Unable to log in, but it's the developer's fault, not yours.");
            console.log("Bad authorization, invalid_request");
          }
        }
        else  {
          this.codeExists(code);
        }
      }
    },
    codeExists(code) {
      code = code.replace(new RegExp("\\" + "#", "gi"), "");

      dbCodesRef.child(code).once("value", snapshot => {
        const codeData = snapshot.val();
        if (codeData !== null) {
          this.getTokenDB(code);
        }
        else  {
          this.getTokenNew(code);
        }
      });
    },
    getTokenDB(code)  {
      dbCodesRef.child(code).once("value", snapshot => {
        const codeData = snapshot.val();
        this.userToken = codeData.token;
      })
      .then(this.getUsername());
    },
    getTokenNew(code)  {
      $.ajax({
        type: "POST",
        url: "https://www.reddit.com/api/v1/access_token",
        beforeSend: xhr => {
          xhr.setRequestHeader("Authorization", "Basic " + window.btoa(client_id + ":" + client_secret));
        },
        data: {grant_type: "authorization_code", code: code, redirect_uri: redirect_uri},
        dataType: "json",
        success: data => {
          this.loggedIn = true;
          this.userToken = data.access_token;
          this.getUsername();
        }
      })
      .then(d => {
        var time = new Date();

        dbCodesRef.child(code).set({
          token: this.userToken,
          timePush: time // TODO: push time and removing after an hour
        })
      });
    },
    getUsername() {
      if (this.userToken !== null) {
        $.ajax({
          type: "GET",
          url: "https://oauth.reddit.com/api/v1/me",
          beforeSend: xhr => {
            xhr.setRequestHeader("Authorization", "bearer " + this.userToken);
          },
          dataType: "json",
          success: data => {
            this.loggedIn = true;
            this.username = data.name;
            console.log(data);
          }
        });
      }
      else  {
        setTimeout(this.getUsername, 100);
      }
    },
    // retrieve user's subscribed subreddits and set a multireddit to emulate user's "front page"
    frontPage() {
      $.ajax({
        type: "GET",
        url: "https://oauth.reddit.com/subreddits/mine/?limit=100", // limited to 100 subreddits, sadly
        beforeSend: xhr => {
          xhr.setRequestHeader("Authorization", "bearer " + this.userToken);
        },
        dataType: "json",
        success: subs => {
          this.subreddit = "";

          subs.data.children.forEach(sub => {
            this.subreddit += sub.data.display_name + "+"
          });

          this.setQuery();
        }
      });
    },
    // retrieve user's comment
    userPosts() {
      this.subUser = "user/";
      this.subreddit = this.username + "/submitted";
      this.setQuery();
    },
    userComments() {
      this.subUser = "user/"
      this.subreddit = this.username + "/comments";
      this.setQuery();
    },
    logout()  {
      $.ajax({
        type: "POST",
        url: "https://www.reddit.com/api/v1/revoke_token",
        beforeSend: xhr => {
          xhr.setRequestHeader("Authorization", "Basic " + window.btoa(client_id + ":" + client_secret));
        },
        data: {token: this.userToken},
        dataType: "json",
        success: data => {
          this.loggedIn = false;
          this.userToken = null;
          dbCodesRef.remove(this.code);
          window.open(redirect_uri, "_self");
        }
      });
    }
    // TODO: make my data available as JSON for others
  },
  mounted: function() {
    this.setQuery();
    // perpetuate login
    this.checkLogin();
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
  margin: 0%;
  width: 100%;
}

#app  {
  height: 100%;
  margin-top: 3%;
  padding: 0;
  width: 100%;
}

.container  {
  margin: 0 0 0 0;
  max-width: 100%;
}

#toggleButtons  {
  width: 100%;
}

#visuals  {
  height: 60%;
  width: 100%;
}

svg {
  border: 1px solid black;
}

.params {
  height: 100%;
  max-height: 100%;
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
