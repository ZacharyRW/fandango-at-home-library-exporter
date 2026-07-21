# Fandango at Home Library Exporter
A python program to login to your vudu account and scrape the names of the movies you own

Before opening, it asks for your Vudu Username and Password

It opens chrome using selenium, must have chrome driver installed

Navigates to Vudu login page

Logs into your account with the credentials you entered

After login it navigates to the My Vudu - Movies section

It scrapes all the movie name data from the page, scrolling while doing so to load the dynamic page content

It then deletes any duplicate names that were scraped

Then it sorts the movie names alphabetically

Asks you what csv file you want the data written to

The program then writes the names to that file  in one column
