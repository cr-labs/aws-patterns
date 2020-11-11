# Time

This is an easy one.

# Agree to use UTC everywhere. Document it. Make your tools work this way.

That's it. That's the tweet, etc.

Patterns:
- when taking time as a CLI argument, consider offering relative times and durations as an option when it makes sense to do so, rather than requiring the user to set a specific time. And make it clear in absolute time args that UTC time is expected. Examples:
  - `--date yesterday` (returns 24 hour period from 00:00 to 23:59 UTC)
  - `--days-ago 3 --duration-days 1` (same as above, but shifted 3 days)
  - `--time-start-utc 2020-11-00 00:00:00 --time-end-utc 2020-11-10 23:59:59`


That last one looks a little unwieldly. More likely what we'd do is all agree to use UTC everywhere, and remove the `-utc` suffix and just use `--time-start` and `--time-end`.


If someone gives you a hard time, here are Reasons:
- commands should produce the same results no matter who runs them or where they are when they run them (exception made for relative commands). Two people running the same command at the same time should get the same result no matter where they are)
- Switching into and out of DST will wreck your productivity because you have to think about it twice a year, and it always happens on a weekend.
- CDNs and other systems out of your control always report UTC because they are literally in every time zone.
- Converting into and out of UTC is unavoidable. So just get it over with and enjoy the freedom.

Helpful things:

- If you use MacOS, install https://github.com/netik/UTCMenuClock by [Netik])https://github.com/netik/UTCMenuClock/commits?author=netik) for a handy menubar UTC clock next to your local time clock.
