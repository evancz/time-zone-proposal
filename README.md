# Time Zone Proposal

Displaying and computing dates accurately requires the [IANA time zone database](https://en.wikipedia.org/wiki/Tz_database). We propose the following API to make this data easily available in browsers:

```
interface TimeZone {
  DOMString getLocalTimeZoneName();
  sequence<TimeZoneEra> getTimeZoneEras(DOMString timeZoneName);
};

dictionary TimeZoneEra {
  long start;
  long end;
  long offset;
  DOMString abbreviation;
};
```

This proposal will benefit page load speed and the reliability of browser based applications. It is also very simple, giving library authors the freedom to create a very broad range of higher-level APIs while keeping the burden on browsers as low as possible.

I recommend checking out [moment-timezone](https://github.com/moment/moment-timezone) or Elm’s [`Time`](https://github.com/elm-lang/core/blob/dev/src/Time.elm) module to see libraries that would use this API directly.

<br>

## Benefit One - Page Load Speed

**Problem:** The raw IANA time zone database is about 300kb when gzipped. Folks must do a bunch of tricks to get this data to users efficiently.

The moment-time project came up with [a custom representation](https://github.com/moment/moment-timezone/blob/develop/data/packed/latest.json) to make the data more compact. When that JSON representation is minified and gzipped, it is about 23kb. This is still not great because:

1. Every website that uses this data must download it separately.
2. The database is updated with new data multiple times per year, so each individual website must keep checking for new versions.

If you pack it with your initial bundle of JS, that means that bundle is just always bigger. If you download it separately, that means you are always making an extra round trip to some server to check for updates.

**Solution:** Make the data accessible through a browser API, removing this download cost from *every* website that uses dates.


## Benefit Two - Reliability

**Problem:** The IANA time zone database is updated multiple times per year. Without the latest version, you may display incorrect times and dates.

This also means that downloading the database once and caching it forever is not viable. Instead, developers must keep an up to date version on their servers, and users must check if a new version is available periodically. The chances of error here are quite large, and it puts an additional operational burden on anyone who wants to work with dates in their application.

How many people get this right? How many people even know about it? And for the developers who do get it right, they end up with the page load problems described above.

**Solution:** Distribute the database with automatic updates of the browser.


## Benefit Three - Simplicity

By keeping the API small, it is useful to a broader range of libraries and languages. The API provides the raw data needed by [moment-timezone](https://github.com/moment/moment-timezone) in JavaScript and for the [`Time`] module in Elm. Both approaches let folks work with dates in a way that makes sense in JS and Elm respectively, but they do so with very different APIs.

So the proposal here could made “easier to use” for JS programmers by adding a bunch of higher-level functions for working with dates, but that would likely make it extremely difficult (or maybe impossible) to create APIs that make sense for different languages and design goals. By keeping this proposal simple, it allows the ecosystem to find great ways of working with dates more organically.

<br>

## Additional Information about Time and Time Zones

Time has many exceptions based on geography, politics, culture, astronomy, etc. I have organized a couple facts along these lines, and I hope will be helpful for folks who would like to vet this proposal in more detail:

  - **Daylight-Saving Time** &mdash; Time zones may choose to shift by an hour for certain parts of the year. Observance [varies greatly](https://en.wikipedia.org/wiki/Daylight_saving_time_by_country) and can change based on the politics and geography of the time zone.

  - **One-Time Events** &mdash; Samoa moved [from UTC-11 to UTC+13](https://en.wikipedia.org/wiki/Time_in_Samoa) in 2011, entirely skipping 30 December. They move to UTC+14 for daylight saving time.
  
  - **Leap Seconds** &mdash; Leap seconds are added about once a year to account for slight changes in Earth‘s orbit. These leap seconds are explicitly not accounted for in POSIX time. If we add one second per year for 60 years, it adds up to one minute, which most casual users of time will struggle to notice in their daily life.

The IANA time zone database tries to track all of these exceptions.
