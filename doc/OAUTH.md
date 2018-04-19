Mod login - be able to delete posts based on words
	oauth.reddit.com/subreddits/mine/moderator
	/api/remove
Users - view personal subreddits "front page", save prefs (on load, stop list), save pref for visualization
	oauth.reddit.com/subreddits/mine/subscriber/.json
	
Log errors:
if data.children.length === 0

https://www.reddit.com/api/v1/authorize?client_id=CLIENT_ID&response_type=TYPE&state=RANDOM_STRING&redirect_uri=URI&duration=DURATION&scope=SCOPE_STRING
* client_id = GeDalotx_uwLig
* response_type = "code"
* state = You should generate a unique, possibly random, string for each authorization request. This value will be returned to you when the user visits your REDIRECT_URI after allowing your app access - you should verify that it matches the one you sent. This ensures that only authorization requests you've started are ones you finish. (You may also use this value to, for example, tell your webserver what action to take after receiving the OAuth2 bearer token)
* redirect_url = https://duke-compsci290-spring2018.github.io/final-project-team-44/
* duration = temporary or permanent
* scope = identity, edit, flair, history, modconfig, modflair, modlog, modposts, modwiki, mysubreddits, privatemessages, read, report, save, submit, subscribe, vote, wikiedit, wikiread
	* mysubreddits
	* modposts
	* submitted
	* comments


user/<username>/submitted
user/<username>/comments