# CS 290 Final project

> Reddit word cloud

Maya Messinger (mm479)  
Started: 13 Apr 18  
Finished: 29 Apr 18
Hours spent: 45

Acessible at [https://duke-compsci290-spring2018.github.io/final-project-team-44/](https://duke-compsci290-spring2018.github.io/final-project-team-44/)

### About the Project
This project is a web application built using Node.js and Vue.js frameworks and [D3 libraries](https://d3js.org/) for the data visualization.
Data is gathered from [Reddit](https://www.reddit.com/dev/api/) in JSON format, then parsed and organized into various data structures for display.
Calls to reddit for login are made using OAuth2, allowing this app access to user's name, front page, and subreddits they mod, offering a 
personalized experience that allows for analysis of their own Reddit behavior.

### Resources used
* D3 data visualization
	* D3 cloud
	* D3 promise
* Stop list of common words from [https://www.ranks.nl/stopwords](https://www.ranks.nl/stopwords)