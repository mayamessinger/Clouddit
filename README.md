# CS 290 Final project

> Reddit word cloud

Maya Messinger  
Started: 13 Apr 18  

### About the Project
This project is a web application built using Node.js and Vue.js frameworks and [D3 libraries](https://d3js.org/) for the data visualization.
Data is gathered from [Reddit](https://www.reddit.com/dev/api/) in JSON format, then parsed and organized into various data structures for display.
Calls to reddit for login are made using OAuth2, allowing this app access to user's name, front page, and subreddits they mod, offering a 
personalized experience that allows for analysis of their own Reddit behavior.

### Resources used
* D3 data visualization
	* D3 cloud
	* D3 promise

### Using Clouddit
* Anyone visiting the site can interact with data from subreddits and users
* Logging in allows a user to cloud their personal front page, their own posts, and their own comments
* Logged in moderators can go to subreddits they moderate, and lock posts and delete posts/comments
	* Useful for quick filtering for specific words, can just ctrl-f for a banned word, view the list of all the posts with that word, and delete those posts