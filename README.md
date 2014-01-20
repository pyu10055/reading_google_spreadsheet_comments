##Retrieving comments from Google Spreadsheet

In one of our project, we are using Google Spreadsheet as a major data source. We are using [Roo Gem](https://github.com/Empact/roo) to access the spreadsheet. Roo actually uses [Google Drive Ruby Gem](https://github.com/gimite/google-drive-ruby) that communicate with [Google Spreadsheet API v3](https://developers.google.com/google-apps/spreadsheets/#working_with_cell-based_feeds).

Everything works pretty well until recently we have to access the comments on individual cells. 
![cell_comment](/comments.png "Cell Comment")

