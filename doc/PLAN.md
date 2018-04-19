PLAN
===

# My application
[Mockup]('mockup.png')

Planning on using Node and Vue
* Word cloud will be a basic word cloud. Duvall mentioned that there are even applications that make word clouds for you, though I do have my own data structure planned.
	* Each word will be a link to a list of all posts with that word in their title (or permalink to comments if data view is of a post instead of a subreddit)
* Bar graph visualization: will display top words in a bar graph. Ideally, bars will be sectioned by subreddits ([like so, only better]('http://www.betterevaluation.org/sites/default/files/2d_StackedColumnChart2.png'))
* Users will have many toggles to filter and curate data (choose subreddit, # of posts, # of words, sort, time, etc.)

My data structure for storing:
```JSON
{
	{
		word: "hello",
		occurrences: 1,
			{
				title: "hello world",
				link: "https://www.reddit.com/...."
			}
		]
	},
	{
		word: "world",
		occurrences: 1,
		posts: [
			{
				title: "hello world",
				link: "https://www.reddit.com/...."
			}
		]
	}
}
```

Data will be grabbed from reddit on an interval, or by a "refresh" button

Efficiency considerations:
General update plan to keep from having to reset all my data just to run it on mostly the same data (does top 24 hours change much in 5 mins? no):
* log unique identifier with most recent post ID
* upon refresh, find how many new posts (n) were made before get to logged unique ident
* run update only on n new posts, call remove only on n outdated posts
* problem: what if rank changes? no longer the "first" post, so unique ident not useful
	* after t=___ arg to query?
	* diff JSON files and only oeprate on diff lines
		* karma and other fields may change, so not perfect diff
	* call update in background, refresh in parallel with static, then push when map is compiled?

Features my site should support:
* default view - probably /r/popular or /r/all hot
* automatic load of basics - top of hour, day, month, year, all time pre-loaded so script doesn't take too long
	* multiple hash maps to save those separately?
	* Vue watch fields to check for updates so I store certain popular arrays?
* user login (to get own front page info)
* user view
* HANDLE MULTIPLE USERS WITH NODE ASYNC

# Reddit's API
Basic query structure: https://www.reddit.com/r/[subreddit]/[sort]/.json?limit=[#]&t=[time]
* /r/[subreddit] is pretty obvious
* [sort] is self-explanatory
	* can take hot, new, controversial, top, rising
* limit=[#] is pretty obvious, # of posts returned
* t=[time] can only be used with sorting by top and controversial
	* can take hour, day, week, month, year, all
Basic post comments structure: https://www.reddit.com/r/[subreddit]/comments/[id]/.json?limit=[#]
* Don't know how this is sorted - does it matter?
User query structure: https://www.reddit.com/user/[user]/.json?limit=[#]
* returns comments and posts, need to find way to differentiate between

To get the fields I want, I'll be grabbing the following data (can test accessing fields with: http://www.jsonquerytool.com/):
```javascript
fetch(url).then(response => response.json())
	.then(posts => {
	    posts.forEach(post => {
	        post.data.children[0].data.title.split(" ").forEach( word =>	{
	        	// see if word is in map
	        		// if yes
	        			occurrences++ and posts.push({title: post.data.children[0].data.title, link: "reddit.com" + post.data.children[0].data.permalink});
	        		// if no
	        			map.push({word: word, occurrences: 1, posts: [{title: post.data.children[0].data.title, link: "https://www.reddit.com" + post.data.children[0].data.permalink}]});
	        }

	    });
	})
	.catch(error => console.log(error))
```
(comment field is body instead of title)

Actual data from example post (top post of all time) as JSON:
```JSON
{
	"kind": "Listing",
	"data": {
		"after": "t3_5gn8ru",
		"dist": 1,
		"modhash": "2dxdnl5ac655bf2666dc7b23c8c85aaacde010b7f2c0100a60",
		"whitelist_status": "all_ads",
		"children": [{
			"kind": "t3",
			"data": {
				"is_crosspostable": true,
				"subreddit_id": "t5_2qh33",
				"approved_at_utc": null,
				"author_flair_text_color": null,
				"mod_reason_by": null,
				"banned_by": null,
				"num_reports": null,
				"author_flair_type": "text",
				"removal_reason": null,
				"thumbnail_width": 140,
				"subreddit": "funny",
				"selftext_html": null,
				"selftext": "",
				"likes": true,
				"suggested_sort": null,
				"user_reports": [],
				"secure_media": null,
				"is_reddit_media_domain": false,
				"saved": false,
				"id": "5gn8ru",
				"banned_at_utc": null,
				"mod_reason_title": null,
				"view_count": null,
				"archived": true,
				"clicked": false,
				"no_follow": false,
				"author": "iH8myPP",
				"num_crossposts": 12,
				"link_flair_text": "Best of 2016 Winner",
				"mod_reports": [],
				"can_mod_post": false,
				"link_flair_richtext": [{
					"e": "text",
					"t": "Best of 2016 Winner"
				}],
				"send_replies": true,
				"pinned": false,
				"score": 283488,
				"approved_by": null,
				"over_18": false,
				"report_reasons": null,
				"domain": "i.imgur.com",
				"hidden": false,
				"preview": {
					"images": [{
						"source": {
							"url": "https://i.redditmedia.com/jzU2DNeq7eTA_cpq4H6Z7Sd5atiVA1JyUx7KwcIJ8N4.jpg?s=e127a2026430e0084080b817723ed23d",
							"width": 890,
							"height": 370
						},
						"resolutions": [{
							"url": "https://i.redditmedia.com/jzU2DNeq7eTA_cpq4H6Z7Sd5atiVA1JyUx7KwcIJ8N4.jpg?fit=crop&amp;crop=faces%2Centropy&amp;arh=2&amp;w=108&amp;s=b58958a1c6f3389eb9d9d9bef0e6b2d0",
							"width": 108,
							"height": 44
						}, {
							"url": "https://i.redditmedia.com/jzU2DNeq7eTA_cpq4H6Z7Sd5atiVA1JyUx7KwcIJ8N4.jpg?fit=crop&amp;crop=faces%2Centropy&amp;arh=2&amp;w=216&amp;s=4a702071331b023f7625b074f3095b70",
							"width": 216,
							"height": 89
						}, {
							"url": "https://i.redditmedia.com/jzU2DNeq7eTA_cpq4H6Z7Sd5atiVA1JyUx7KwcIJ8N4.jpg?fit=crop&amp;crop=faces%2Centropy&amp;arh=2&amp;w=320&amp;s=4678fea2b4a1d4c43fc4296a08ecb945",
							"width": 320,
							"height": 133
						}, {
							"url": "https://i.redditmedia.com/jzU2DNeq7eTA_cpq4H6Z7Sd5atiVA1JyUx7KwcIJ8N4.jpg?fit=crop&amp;crop=faces%2Centropy&amp;arh=2&amp;w=640&amp;s=c0e4e8503451b2d6971d7a8feac8e463",
							"width": 640,
							"height": 266
						}],
						"variants": {},
						"id": "y2rlIOoU9df4PrvL34ZKpC4hLj-hHtvIubIxqP0s8Zg"
					}],
					"reddit_video_preview": {
						"fallback_url": "https://v.redd.it/ev41i1xcq5q01/DASH_1_2_M",
						"height": 266,
						"width": 640,
						"scrubber_media_url": "https://v.redd.it/ev41i1xcq5q01/DASH_600_K",
						"dash_url": "https://v.redd.it/ev41i1xcq5q01/DASHPlaylist.mpd",
						"duration": 36,
						"hls_url": "https://v.redd.it/ev41i1xcq5q01/HLSPlaylist.m3u8",
						"is_gif": true,
						"transcoding_status": "completed"
					},
					"enabled": false
				},
				"thumbnail": "https://b.thumbs.redditmedia.com/yeLMMXr9vghtaz26xZ1A_PCg-vk4_KjtnHNlQ-NzkxA.jpg",
				"hide_score": false,
				"edited": false,
				"link_flair_css_class": "",
				"author_flair_richtext": [],
				"author_flair_css_class": null,
				"contest_mode": false,
				"gilded": 13,
				"locked": false,
				"downs": 0,
				"brand_safe": true,
				"subreddit_subscribers": 19221007,
				"secure_media_embed": {},
				"media_embed": {},
				"post_hint": "link",
				"author_flair_text": null,
				"visited": false,
				"can_gild": true,
				"thumbnail_height": 58,
				"parent_whitelist_status": "all_ads",
				"name": "t3_5gn8ru",
				"spoiler": false,
				"permalink": "/r/funny/comments/5gn8ru/guardians_of_the_front_page/",
				"subreddit_type": "public",
				"whitelist_status": "all_ads",
				"stickied": false,
				"created": 1480988474.0,
				"url": "http://i.imgur.com/OOFRJvr.gifv",
				"link_flair_type": "richtext",
				"quarantine": false,
				"rte_mode": "markdown",
				"created_utc": 1480959674.0,
				"subreddit_name_prefixed": "r/funny",
				"ups": 283488,
				"media": null,
				"link_flair_text_color": "dark",
				"is_original_content": false,
				"author_flair_background_color": null,
				"num_comments": 5064,
				"is_self": false,
				"title": "Guardians of the Front Page",
				"mod_note": null,
				"is_video": false,
				"distinguished": null
			}
		}],
		"before": null
	}
}
```