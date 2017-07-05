# Local Times Proposal

  - Computing local times accurately requires the [IANA time zone database](https://en.wikipedia.org/wiki/Tz_database)
  - The database is quite large by web standards (+300kb gzipped)
  - We recommend that browsers:

  	  1. **Distribute this database**, adding to the database as necessary as part of their existing automatic update mechanism.
  	  2. **Provide a low-level JavaScript API to access this data.** This will allow the creation of high-level date libraries that make a variety of choices. For example, Elm and JS will almost certainly want to expose this data differently.


## Why do we need this database?

Let’s start by asking **“how do you convert a POSIX time to a local time?”**

It turns out it is very complicated! Some important factors are:

  - [**Leap Seconds**](https://en.wikipedia.org/wiki/Leap_second) &mdash; The Earth’s rotation speed varies in response to climatic and geological events, so extra seconds are added unpredictably by the International Earth Rotation and Reference Systems Service (IERS) usually about six months in advance.

  - **Daylight Saving Time** &mdash; Time zones may choose to add an hour for certain parts of the year. Observance [varies greatly](https://en.wikipedia.org/wiki/Daylight_saving_time_by_country) and can change based on the politics and geography of the time zone.

  - **One-Time Events** &mdash; Samoa moved [from UTC-11 to UTC+13](https://en.wikipedia.org/wiki/Time_in_Samoa) in 2011, entirely skipping 30 December. They move to UTC+14 for daylight saving time.

There are many other oddities that make it very difficult to compute local times accurately!

In fact, there are so many oddities that **anyone who wants accurate local times relies on [this database of exceptions](https://www.iana.org/time-zones).**


## Why do we need it in the browser?

Why not just download the timezone data if you need it?

Well, **this timezone database is quite large** and everyone on the internet who wants accurate local times needs it. As a result, a variety of bad outcomes are very likely:

  1. **Ignore the Problem** &mdash; Very few folks (1) know about this problem and (2) are able to do the technical work to distribute this data and compute times accurately.

  2. **Naive Solution** &mdash; Just distribute the 300kb+ database to everyone who visits your website. Now they get good local times, but your site is just slower for them.

  3. **Fancy Solution** &mdash; Perhaps you really care about getting times right, but you do not want to add 300kb+ to every single page visit. Maybe you have some engineers figure out how to (1) detect the timezone of the client and (2) distribute the relevant subset of the database.

I suspect that a majority of folks choose path 1, and for the folks who are aware that paths 2 and 3 even exist, I suspect a majority still choose path 1.

If people decide to go path 3 today, they must pay a couple important costs. (1) They must spend development time. (2) They may get things wrong, resulting in bad local times. (3) They may need to load the times lazily, so the first time conversion may need an HTTP request, resulting in weird user experiences. (4) They must accept that they must serve larger bundles.

Point is, the best case is not great, and **if this data was available in the browser, people could default into getting local times right without any asset bloat, risky code, dev time, etc.**


## Draft API

This is where the draft API will live.