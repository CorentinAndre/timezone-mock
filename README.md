timezone-mock
================

A JavaScript library to mock the local timezone.

This module is useful for testing that code works correctly when run in
other timezones, especially those which have Daylight Saving Time if the
timezone of your test system does not.

When `register` is called, it replaces the global Date constructor with
a mocked Date object which behaves as if it is in the specified timezone.

Note: Future timezone transitions are likely to change due to laws, etc.  Make
sure to always test using specific dates in the past. The timezone data used by
`timezone-mock 1.0.4+` should be up accurate for all times through the end of 2018.

Note: Node v8.0.0 changed how the string "YYYY-MM-DDTHH:MM:SS" is interpreted.
It was previously interpreted as a UTC date, but now is a local date. If your
code is using dates of this format, results will be inconsistent.  timezone-mock
treats them as a local date, so that it behaves consistently with new versions
of Node, but that means if you run the tests in here on old versions of node,
or use the mock on old versions of node, the tests may not be accurate (just
for parsing dates in the aforementioned format).


Usage Example
=============

```javascript
var assert = require('assert');
var timezone_mock = require('timezone-mock');

function buggyCode() {
  // This function is potentially a bug since it's interpreting a string in
  // the local timezone, which will behave differently depending on which
  // system it is ran on.
  return new Date('2015-01-01 12:00:00').getTime();
}
var result_local = buggyCode();
timezone_mock.register('US/Pacific');
var result_pacific = buggyCode();
timezone_mock.register('US/Eastern');
var result_eastern = buggyCode();
assert.equal(result_local, result_pacific); // Might fail
assert.equal(result_pacific, result_eastern); // Definitely fails

```

API
===
* `timezone_mock.register(timezone)` - Replace the global Date object with a mocked one for
the specified timezone.  Defaults to 'US/Pacific' if no timezone is specified.
* `timezone_mock.unregister()` - Return to normal Date object behavior
* `timezone_mock._Date` - access to the original Date object for testing

Supported Timezones
===================
Currently supported timezones are:
* US/Pacific
* US/Eastern
* Brazil/East
* UTC

I found that testing on these three were enough to ensure code worked in
all timezones (import factor is to test on a timezone with Daylight Saving
Time if your local timezone does not).  Brazil/East has the unique characteristic
of having the DST transition happen right at midnight, so code that sets a Date
object to midnight on a particular day and then does operations on that Date
object is especially vulnerable in that timezone.

Status
======

Most Date member functions are supported except for some conversions to
locale-specific date strings.

With non-DST timezones, it should behave identically to the native Javascript
Date object.  With DST timezones, it may sometimes behave slightly differently
when given an ambiguous date string (e.g. "2014-11-02 01:00:00" in "US/Pacific",
is treated as 1AM PDT instead of 1AM PST - same clock time, utc timestamp off by
an hour).
