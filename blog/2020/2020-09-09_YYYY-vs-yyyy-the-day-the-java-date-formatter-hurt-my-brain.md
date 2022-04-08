---
title: YYYY vs yyyy - The day the Java Date Formatter hurt my brain 
published: true
description: 
tags: Java, JSON, ISO, Dates
cover_image: https://dev-to-uploads.s3.amazonaws.com/i/h5p1zz5rumotbpfnob3j.png
---

# Intro
Dates in programming are hard. The ISO-8601 Date standard made them easier but to be entirely honest I'm not ISO-8601 Certified so I'm not very good at my job. What I do know though is the very important difference between `YYYY` and `yyyy` when formatting dates.

# The Difference between y and Y
According to the DateTimeFormatter Java Docs (which implements the ISO-8601 specification)
 - y (lowercase) is year
 - Y (uppercase) is 'week-based-year'.

This difference will cause your code to work perfectly fine, except for when dealing with dates at the very end of some years. 

Here is an example that I had the absolute PLEASURE of dealing with last year. For the date 31/12/2019 (December 31st 2019), `yyyy` will output 2019 but `YYYY` will output 2020. The reason for this is the week that the 31st of December falls in technically is the first week of 2020. This issue can be a pain to notice and debug.
In my case, 31/12/2019 was being sent from the frontend, being stored correctly in the DB, but was turning into 31/12/2020 when being returned and no one had any idea why. Our default JSON date formatter was the culprit, using `YYYY` where it should have been using `yyyy` (some clumsily copied and pasted quick solution by myself ofcourse).

# Example

## Declare formatters
```java
// y (Lowercase) 
DateTimeFormatter formatteryyyy = DateTimeFormatter.ofPattern("yyyy-MM-dd");
// Y (Uppercase)
DateTimeFormatter formatterYYYY = DateTimeFormatter.ofPattern("YYYY-MM-dd");
```
## When converting from Date to String
```java
// Contains value from Database (December 31st 2019) 
LocalDateTime dateTime;

// String value will be 2019-12-31
String yyyy = dateTime.format(formatteryyyy);

// String value will be 2020-12-31
String YYYY = dateTime.format(formatterYYYY);
```
## When convert from String to Date
```java
// Actual date value will be 2020-12-31
LocalDateTime parsedDateyyyy = LocalDateTime.parse("2020-12-31", formatteryyyy); 

// Actual date value will be 2019-12-31
LocalDateTime parsedDateYYYY= LocalDateTime.parse("2020-12-31", formatterYYYY);  
```

# Conclusion 
In 99% of use cases, the following is true:
 - yyyy good
 - YYYY bad

