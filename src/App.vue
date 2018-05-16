<template>
  <div id="app" class="container">
    <devInfo></devInfo>
    <div id="toggleButtons" class="row">
      <graphToggles class="col-9" :lastUpdated="lastUpdated" :weightChosen="weightOption" :weightOptions="weightOptions" :title="selected" @weigh="weigh($event)" @refresh="ready()">
      </graphToggles>
      <login class="col-3" v-if="!loggedIn" @login="login()"></login>
      <userOptions class="col-3" v-if="loggedIn" :username="username" @frontPage="frontPage()" @userPosts="userPosts()" @userComments="userComments()" @logout="logout()"></userOptions>
    </div>
    <div class="row" id="visuals">
      <myCloud class="col-9" id="cloud">
      </myCloud>
      <div class="params col-3">
        <subUser :ru="subUser" v-model="subUser"></subUser>
        <subreddit :subreddit="subreddit" v-model="subreddit"></subreddit>
        <sort :sort="sort" :sorts="sorts" :ru="subUser" v-model="sort"></sort>
        <limit :limit="limit" v-model="limit"></limit>
        <timeO :time="time" :times="times" :sort="sort" v-model="time"></timeO>
        <input class="param" type="button" v-on:click="setQuery" value="Visualize" /><br />
        <button class="param" id="export" v-on:click="exportData">Export current data</button>
        <stopList :stopList="excluded" :oldStop="prevExcluded" @addBackWord="addBackWord($event)">
        </stopList>
      </div>
    </div>
    <div class="row">
      <entries class="entries col-9" :word="wordSelected" :entries="entriesSelected" :isMod="isMod" @visComs="entryReplies($event)" @modLock="modLock($event)" @modRemove="modRemove($event)">
      </entries>
    </div>
  </div>
</template>

<script>
// TODO: secure reading and writing
// TODO; remove api keys and variables from raw js
import excluded from "./assets/stopList2.json";
import $ from "jquery";
import bootstrap from "bootstrap";
import * as firebase from "firebase";
import * as d3 from "d3";
import * as cloud from "d3-cloud";
import * as promise from "d3.promise";

import DevInfo from "./components/DevInfo.vue";
import Login from "./components/Login.vue";
import UserOptions from "./components/UserOptions.vue";
import GraphToggles from "./components/GraphToggles.vue";
import MyCloud from "./components/Cloud.vue";
import Entries from "./components/Entries.vue";
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

var domParser = new DOMParser();
const redditDomain = "https://www.reddit.com/";
const client_id = "GeDalotx_uwLig";
const client_secret = "0WCPPtvbnnqhYKUIDHfN4Wdns7M";
const redirect_uri = "https://duke-compsci290-spring2018.github.io/final-project-team-44/";

export default {
  name: "app",
  data() {
    return {
      loggedIn: false,
      username: null,
      isMod: false,
      userCode: null,
      userToken: null,
      subUser: "r/",
      subreddit: "all",
      sort: "hot",
      sorts: ["best", "hot", "new", "controversial", "top", "rising"],
      limit: 50,
      time: "",
      times: ["hour", "day", "week", "month", "year", "all"],
      redditUrl: redditDomain + this.subUser + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time,
      map: [],
      excluded: excluded.words,
      prevExcluded: [],
      selected: null,
      wordSelected: null,
      entriesSelected: [],
      highestOccurrences: 0,
      highestUpvotes: 0,
      lastUpdated: this.newTime(),
      weightOption: "upvotes",
      weightOptions: ["upvotes", "occurrences"]
    }
  },
  components: {
    DevInfo,
    Login,
    UserOptions,
    GraphToggles,
    MyCloud,
    Entries,
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
      switch (this.subUser) {
      	case "user/":
      		this.redditUrl = redditDomain + this.subUser + this.subreddit + "/.json?limit=" + this.limit;
      		break;
      	case "r/":
      		this.redditUrl = redditDomain + this.subUser + this.subreddit + "/" + this.sort + "/.json?limit=" + this.limit + "&t=" + this.time;
      		break;
      }

      this.setEntrySelected(this.subreddit);

      setTimeout(this.ready, 100);
    },
    // call make map
    ready() {
      this.newTime();
      this.map = [];
      this.wordSelected = null;
      this.entriesSelected = [];
      this.isMod = false;
      this.highestOccurrences = 0;
      this.highestUpvotes = 0;
      this.checkModStatus();
      this.getEntriesData(this.redditUrl).then(this.makeCloud());
    },
    // make header to clarify what you're visualizing
    setEntrySelected(entry)  {
      this.selected = entry;
      if (this.selected.length > 250) {
        this.selected = this.selected.substring(0, 250) + "...";
      }
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
    getEntriesData(url)  {
      return promise.json(url, entries => {
        if (entries === null) {
          this.emptySub();
          return;
        }

        this.parseEntries(entries);
      });
    },
    parseEntries(entries)  {
      if (entries.length > 1) {
        entries = entries[1];
      }

      entries.data.children.forEach(entry => {
        switch (entry.kind)	{
        	// post
        	case "t3":
        		entry.data.title.split(" ").forEach( word =>  {
	            word = this.prettify(word);

	            if (this.isExcluded(word)) {
	              return;
	            }
	            
	            this.addToMap(word, entry);
	          });
	          break;
	        // comment
	        case "t1":
	        	entry.data.body.split(" ").forEach( word =>  {
	            word = this.prettify(word);

	            if (this.isExcluded(word)) {
	              return;
	            }
	            
	            this.addToMap(word, entry);
	          });
	          break;
        }
      });
    },
    // alert about bad subreddit call
    emptySub()  {
      alert("Oh no! Looks like /r/" + this.subreddit + " doesn't exist!");
    },
    // check if word should be added to stored data, then do so if it should
    addToMap(word, entry)  {
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
        this.map[i].entries.forEach(loggedPost => {
          if (loggedPost.link === redditDomain + entry.data.permalink) {
            newPost = false;
          }
        });

        if (newPost) {
          this.map[i].upvotes += entry.data.ups;
          if (this.map[i].upvotes > this.highestUpvotes) {
            this.highestUpvotes = this.map[i].upvotes;
          }

          if (entry.data.title !== undefined) {
            this.map[index].entries.push({title: domParser.parseFromString(entry.data.title, "text/html").body.textContent, 
                                        subreddit: entry.data.subreddit_name_prefixed,
                                        name: entry.data.name,
                                        link: redditDomain + entry.data.permalink,
                                        upvotes: entry.data.ups,
                                        nsfw: entry.data.over_18});
          }
          else if (entry.data.body !== undefined)  {
            this.map[index].entries.push({title: domParser.parseFromString(entry.data.body, "text/html").body.textContent,
                                        subreddit: entry.data.subreddit_name_prefixed,
                                        name: entry.data.name,
                                        link: redditDomain + entry.data.permalink,
                                        upvotes: entry.data.ups});
          }
        }
      }
      else  {
        if (entry.data.ups > this.highestUpvotes) {
            this.highestUpvotes = entry.data.ups;
          }

        if (entry.data.title !== undefined) {
          this.map.push({word: word,
                        occurrences: 1,
                        upvotes: entry.data.ups,
                        entries:
                          [{title: domParser.parseFromString(entry.data.title, "text/html").body.textContent,
                            subreddit: entry.data.subreddit_name_prefixed,
                            name: entry.data.name,
                            link: redditDomain + entry.data.permalink,
                            upvotes: entry.data.ups,
                            nsfw: entry.data.over_18}]});
        }
        else if (entry.data.body !== undefined)  {
          this.map.push({word: word,
                        occurrences: 1,
                        upvotes: entry.data.ups,
                        entries:
                          [{title: domParser.parseFromString(entry.data.body, "text/html").body.textContent,
                            subreddit: entry.data.subreddit_name_prefixed,
                            name: entry.data.name,
                            link: redditDomain + entry.data.permalink,
                            upvotes: entry.data.ups}]});
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
        this.ready();
      }
    },
    // update word size mapping to reflect weight option
    decideWeight() {
      var words;

      switch (this.weightOption)	{
      	case "upvotes":
      		words = this.map
	          .map(d => {
	            return {text: d.word,
	                    occurrences: d.occurrences,
	                    upvotes: d.upvotes,
	                    entries: d.entries,
	                    size: d.upvotes / this.highestUpvotes * 100};
	          });
	        break;
	      case "occurrences":
	      	words = this.map
		        .map(d => {
		          return {text: d.word,
		                  occurrences: d.occurrences,
		                  upvotes: d.upvotes,
		                  entries: d.entries,
		                  size: d.occurrences / this.highestOccurrences * 100};
		        });
		      break;
      }

      return words;
    },
    // calculate font sizes - relation to max instead of hard-coded
    calcTextSize(value)  {
      switch (this.weightOption)	{
      	case "upvotes":
      		return (value/this.highestUpvotes) * 100;
      	case "occurrences":
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
          this.displayEntries(d.entries);
        });
    },
    // when word is clicked, show entries with word in them
    displayEntries(entries) {
      this.entriesSelected = [];
      entries.forEach(entry => {
        this.entriesSelected.push(entry);
      });
    },
    // visualize the replies to a post or comment
    entryReplies(post)  {
      this.setEntrySelected(post.title);

      this.redditUrl = post.link + ".json?limit=" + this.limit + "&t=" + this.time;
      this.ready();
    },
    // lock a post
    modLock(post) {
      var conf = confirm("Confirm lock of:\n" + post.title);

      if (conf) {
        $.ajax({
          type: "POST",
          url: "https://oauth.reddit.com/api/lock",
          beforeSend: xhr => {
            xhr.setRequestHeader("Authorization", "bearer " + this.userToken);
          },
          data: {id: post.name},
          dataType: "json",
          success: data => {
            alert(post.title + " locked.");
          },
          error: d => {
            alert("Could not lock " + post.title);
          }
        });
      }
    },
    // remove a post or comment
    modRemove(entry) {
      var conf = confirm("Confirm removal of:\n" + entry.title);

      if (conf) {
        $.ajax({
          type: "POST",
          url: "https://oauth.reddit.com/api/remove",
          beforeSend: xhr => {
            xhr.setRequestHeader("Authorization", "bearer " + this.userToken);
          },
          data: {id: entry.name, spam: false},
          dataType: "json",
          success: data => {
            alert(entry.title + " removed");
          },
          error: d => {
            alert("Could not remove " + entry.title);
          }
        });
      }
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

        args.forEach(param => {
          if (param.match(errorEXP) !== null) {
            error = param.match(errorEXP)[1];
          }
          else if (param.match(stateEXP) !== null) {
            state = param.match(stateEXP)[1];
          }
          else if (param.match(codeEXP) !== null) {
            this.userCode = param.match(codeEXP)[1];
            this.userCode = this.userCode.replace(new RegExp("\\#", "gi"), "");
          }
        });

        if (error !== null) {
          console.log(error + " for " + state);
          switch (error)	{
          	case "access_denied":
          		alert("Login request was not approved");
          		break;
          	case "unsupported_response_type":
          		alert("Unable to log in, but it's the developer's fault, not yours.");
            	console.log("response_type not token");
            	break;
            case "invalid_scope":
            	alert("Unable to log in, but it's the developer's fault, not yours.");
            	console.log("invalid_scope");
            	break;
            case "invalid_request":
            	alert("Unable to log in, but it's the developer's fault, not yours.");
            	console.log("Bad authorization, invalid_request");
            	break;
          }
        }
        else  {
          this.codeExists();
        }
      }
    },
    // if code exists as entry in firebase, determines if need to get new token or token from db
    codeExists() {
      dbCodesRef.child(this.userCode).once("value", snapshot => {
        const codeData = snapshot.val();
        if (codeData !== null) {
          this.getTokenDB();
        }
        else  {
          this.getTokenNew();
        }
      });
    },
    // get previously-authorized authorization token from firebase
    getTokenDB()  {
      dbCodesRef.child(this.userCode).once("value", snapshot => {
        const codeData = snapshot.val();
        this.userToken = codeData.token;
      })
      .then(this.getUsername());
    },
    // get reddit authorization_token from reddit, does not exist in database
    getTokenNew()  {
      $.ajax({
        type: "POST",
        url: "https://www.reddit.com/api/v1/access_token",
        beforeSend: xhr => {
          xhr.setRequestHeader("Authorization", "Basic " + window.btoa(client_id + ":" + client_secret));
        },
        data: {grant_type: "authorization_code", code: this.userCode, redirect_uri: redirect_uri},
        dataType: "json",
        success: data => {
          this.loggedIn = true;
          this.userToken = data.access_token;
          this.getUsername();
        }
      })
      .then(d => {
        var timePut = Date.now()/1000;

        dbCodesRef.child(this.userCode).set({
          token: this.userToken,
          timePush: timePut.toString()
        });
      });
    },
    // get username of logged on user after login or refresh
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
          }
        });
      }
      else  {
        setTimeout(this.getUsername, 100);
      }
    },
    // check if user is a mod of a subreddit, to be able to display mod options
    checkModStatus()  {
      if (this.mightBeSubreddit()) {
        $.ajax({
          type: "GET",
          url: "https://www.reddit.com/r/" + this.subreddit + "/about/moderators.json",
          dataType: "json",
          success: mods => {
            mods.data.children.forEach(mod => {
              if (mod.name === this.username) {
                this.isMod = true;
              }
            });
          },
          error: d => {
            console.log("not a subreddit");
          }
        });
      }
    },
    // mif string may be subreddit (as opposed to multireddit or user
    mightBeSubreddit()  {
      var stc = this.subreddit;
      stc.toLowerCase();

      if (this.subUser === "user/") {
        return false;
      }
      else if (stc.includes("+")) {
        return false;
      }
      else if (stc === "all" || this.subreddit === "popular") {
        return false;
      }
      else  {
        return true;
      }
    },
    // retrieve user's subscribed subreddits and set a multireddit to emulate user's "front page"
    frontPage() {
      $.ajax({
        type: "GET",
        url: "https://oauth.reddit.com/subreddits/mine/?limit=100",
        beforeSend: xhr => {
          xhr.setRequestHeader("Authorization", "bearer " + this.userToken);
        },
        dataType: "json",
        success: subs => {
          this.subreddit = "";

          subs.data.children.forEach(sub => {
            this.subreddit += sub.data.display_name + "+";
          });

          this.setQuery();
        }
      });
    },
    // retrieve user's posts to visualize
    userPosts() {
      this.subUser = "user/";
      this.subreddit = this.username + "/submitted";
      this.setQuery();
    },
    // retrieve user's comments to visualize
    userComments() {
      this.subUser = "user/";
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
          dbCodesRef.child(this.userCode).remove();
          this.loggedIn = false;
          this.username = null;
          this.isMod = null;
          this.userCode = null;
          this.userToken = null;
          window.open(redirect_uri, "_self");
        }
      });
    },
    // clears old (expired) tokens from Firebase (expire after 1 hour)
    clearOldCodes() {
      var toRemove = [];

      dbCodesRef.on("value", snapshot => {
        snapshot.forEach(item => {
          if ((Date.now()/1000) - parseInt(item.node_.children_.root_.left.value.value_) >= 3600) {
            toRemove.push(item.key);
          }
        });
      });

      setTimeout(
        d => {
          toRemove.forEach( key => {
            dbCodesRef.child(key).remove();
          });
        },
      3000);
    },
    // make my data available as JSON for others
    exportData()  {
      var myWindow = window.open("", "Clouddit word map");
      myWindow.document.write(JSON.stringify(this.map));
    }
  },
  mounted: function() {
    this.clearOldCodes();
    this.setQuery();
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
  margin-top: 1%;
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

#export {
  border: none;
  margin-top: 2%;
  padding: 0 1.5%;
}
</style>
