---
layout: post
title: "Syncing Calendars"
date: 2017-03-09
---

I work in the Frederick Seitz Materials Research Laboratory (MRL) where researchers create devices smaller than your fingernail. To accomplish this, we write patterns on substrates, usually made of thermally grown silicon or glass (among a myriad other materials), using electrons or light to create a pattern, which is then developed in a process exactly like [photography](https://en.wikipedia.org/wiki/Photography). After that, we employ some combination of etching and material deposition to fill the pattern.

In a building where hundreds of researchers work on their individual projects, it is necessary to organize who can use a piece of equipment during some set times. The MRL staff have kindly developed an online scheduling system to create some order.

Below is an example of what the schedule looks like for E-beam Evaporator 4:
&nbsp;

![E-beam 4 Schedule](/assets/images/2017-03-05-mrl-schedule.png)
&nbsp;

A researcher can see which other users are using a piece of equipment at a given time, and if given sufficient privileges, he or she can reserve time. Let us suppose that a researcher reserved a time slot. Then the home page of the researcher displays all of the sessions he or she signed up for.
&nbsp;

![Scheduled sessions](/assets/images/2017-03-05-scheduled-sessions.png)
&nbsp;

This is already a great service that the MRL staff provides, but what if you would like to sync up your schedule with your Google Calendar (or other calendar of choice)? 
&nbsp;

![Initial Google Calendar](/assets/images/2017-03-05-initial-cal.png)
&nbsp;

It is admittedly quite trivial to look between the scheduled sessions and your Google Calendar and just click and drag your mouse to create an event, but a common problem is that schedules are quite dynamic because samples do not work out or users cancel their schedule, opening up new time slots that you would like to take. It can be difficult to track all of those changes, so I implemented a new way to show these changes on my Google Calendar.

The basic idea is that I scraped information from the table of scheduled sessions and accessed the Google Calendar using its API. Unfortunately, the MRL staff have not developed the scheduling system far enough to supply an API to access schedule data, so I scraped the information by looking at the source code. Not ideal, of course, as it is absolutely not robust against changes to the webpage that displays the tables.

To start, I implemented [mechanicalsoup](https://github.com/hickford/MechanicalSoup) to log into the calendar system:
&nbsp;

![Log In screen](/assets/images/2017-03-05-mrl-login.png)
&nbsp;

As it turns out, Python really does make everything easy:

```
browser = mechanicalsoup.Browser()
login_page = browser.get(website)
login_form = login_page.soup.select('form')[0]

login_form.select('#usrID')[0]['value'] = user
login_form.select('#usrPswd')[0]['value'] = password

home_page = browser.submit(login_form, login_page.url)
```

This returns the page which contains the table of scheduled sessions. The code takes all the rows from that particular table, cleans it up, and returns a list of of three items: the equipment name, the start time, and the end time.

The last part is a routine which first clears the schedule then reinserts all the events, which may or may not be the best solution to replacing the schedule. I think that in the end, it probably would take about the same running time to check before deleting vs. completely deleting then reinserting everything.

Using the correct datetime object to send into the event body (which contains information like equipment name and start and end times) was one of the trickier parts, but it turned out that it required just a simple change to the pattern detection string:

```
def extractDatetime(datetime_string):
	pattern = '%m/%d/%Y %I:%M:%S %p'
	datetime_output = datetime.datetime.strptime(datetime_string,pattern)
```

There are some other issues with dealing with RFC3339 formatting and making sure you're in the correct time zone, but aside from that the workflow is relatively simple. The final product looks like this:
&nbsp;

![Final Google Calendar](/assets/images/2017-03-05-final-cal.png)
&nbsp;

At this point, I'm hoping to graduate soon, so I don't really know how much more I'm willing to work on this, but you could always check out my code on github and make more improvements. One thing I know that can and should absolutely be changed is the hard-coding that is used. In principle, the variables user and password ought to take information from an external file (like how the GCalHelper file accesses `client_secret.json`) so that information is kept safe and the script is not tied to a specific individual. Like I mentioned above, perhaps the MRL staff could implement an API to help the MRL users access schedule data much more easily. You could imagine that doing so could also aid in the development of a mobile app, though I think that is probably still a bit naive and trivial if the MRL staff just release a web version of the scheduling website.