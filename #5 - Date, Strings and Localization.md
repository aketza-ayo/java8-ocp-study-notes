# Working with Dates and Times
In Java 8, Oracle revamped how we work with dates and times. You can still write code the old way but those classes aren't in the exam. You need to import ```import java.time.*;``` package.

## Creating Dates and Times
When working with dates and times the first thing you do is to decide how much information you need. The exam gives you four choices:
- **LocalDate**: Contains just a date - no time and no time zone. A good example of LocalDate is your birthday this year.

- **LocalTime**: Contains just time - no date and no time zone. A good example of LocalTime is midnight.

- **LocalDateTime**: Contains both a date and time but no time zone.

- **ZonedDateTime**: Contains a date, time and time zone.

Oracle recommends avoiding time zones unless you really need them. Try to act as if everyone is in the same time zone when you can. You obtain date and time instances suing a static method:

```java
System.out.println(LocalDate.now());              //2018-01-21
System.out.println(LocalTime.now());              //11:39:40.456
System.out.println(LocalDateTime.now());          //2018-01-21T11:39:40.456
System.out.println(ZonedDateTime.now());          //2018-01-21T11:39:40.456-04:00[America/New_York]
```

The key in this of information is the output. Java use T to separate date and time when convertine LocalDateTime to string. Finally the forth line adds the time zone offset and time zone. New York is four time zones awayfrom Greenwich Mean Time(GMT). Greenwich Mean Time is a time zone in Europe that is used as time zone zero when discussing offsets. You might have also heard of Coordinated Universal Time, which is a time zone standard. It is abbreviated as a UTC, as a compromise between the English and French names. UTC uses the same time zone zero as GMT. Let's make sure you understand how UTC works. We include names of the time zones in the example to make them easier to picture. The exam will give you the UTC offset. You are not expected to memrize any time zones.
First lets try to figure out how far apart these moments are in time. Notice how India has half-hour offset, not a full hour. To approach a problem like this, you substract the time zone from the time. This gives you the GMT equivalent of this time.

```
2018-01-21T07:50+02:00[Europe/Paris]      //GMT 2018-01-21  05:50
2018-01-21T06:50+05:30[Europe/Paris]      //GMT 2018-01-21  01:20
```

After converting to GMT, you can see that the Paris time is four ahd a half hours behind the Kolkata time

:yin_yang: The time zone offset can be listed in different ways: +02:00, GMT+2 and UTC+2 all eman the same thing. You might see any of them on the exam.

Lets ty another example, this time with GMT. Remember that you need to add when sunstracting a negative number.

```java
2018-01-21T07:50 GMT-04:00  //GMT 2018-01-21 11:50
2018-01-21T04:50 GMT-07:00  //GMT 2018-01-21 11:50  
```

For this example, both moments in time are the same. The eastern US time zone is three hours ahead of the pacific US time zone. 

Try the example below. You need to be able to perform this math on the exam:

```
2018-01-21T07:50-07:00[US/Pacific]
2018-01-21T07:50+05:30[Asia/Kolkata]
```
The answer is 12 and a half hours. The first instance in time is GMT 14:50. The second is GMT 02:20. The way I figure this one out by doing the opposite operati0on of what is shown in the offset. That is, when you see a minus sign you add into the time and when you see a plus sign you substract into the time.

Moving on, let's create just a date with no time. Both of these examples create the same date.

```java
LocalDate date1 = LocalDate.of(2018, Month.JANUARY, 21);
LOcalDate date2 = LocalDate.of(2018, 1, 21);
```

Both pass in the year, month and date. Although it is good to use the Month constant you can pass the int number of the month directly. Just use the natural number of the month. Month is an Enum, remember that an Enum is not an int and cannot be compared to one, for example:

```java
Month month = Month.JANUARY;
boolean b1 = month == 1;              // DOES NOT COMPILE
boolean b2 = month == Month.APRIL;    // false
```
In the similar way as with Date, when creating a time you can choose how detailed you want to be. You can specify just the hour and minute, or you can include the number of seconds. You can even include nanoseconds if you want to be very precise. 

```java
LocalTime time1 = LocalTime.of(6, 15);                  // hour and minute
LocalTime time2 = LocalTime.of(6, 15, 30);              // + second
LocalTime time3 = LocalTime.of(6, 15, 30, 200);         // + nanosecond
```

you can also combine dates and times into one object:

```java
LocalDateTime dateTime1 = LocalDateTime.of(2018, Month.JANUARY, 20, 6, 15, 30);
LocalDateTime dateTime2 = LocalDateTime.of(date1, time1)
```
The first line shows how to specify all of the information about the LocalDateTime right in the same line. The second line shows how you can create LocalDate and LocalTime objects separately first asnd then conbine them to create a LocalDateTime object. 

In order to create a ZonedDateTime, we first need to get the desired time zone. We will use US/Eastern in our example:

```java
ZoneId zone = ZoneId.of("US/Eastern");
ZonedDateTime zoned1 = ZonedDateTime.of(2018, 1, 21, 6, 15, 30, 200, zone);
ZonedDateTime zoned2 = ZonedDateTime.of(date1, time1, zone);
ZonedDateTime zoned3 = ZonedDateTime.of(dateTime1, zone);
```

We start by getting the time zone object. Then we use of the three approaches to create the ZonedDateTime. The first passes all of the fields individually. We dont recoomend this approach - there are too many numbers and is hard to read. A better approach is to pass a LocalDate object and a LocalTime object or a LocalDateTime object.Although there are other ways of creating a ZonedDateTime, you only need to know these three. Notice that there isn't a way to pass in the Month enum. This seems like an oversight from the API designers and something that will be fixed in future versions of Java.

Did you totice that we did not use a constructor in any of the examples? The date and time cl;asses have private constructors to force you use the factory static methods. The examcreates might throw you somethig like this:

```java
LocaDate date = new LocalDate();      //DOES NOT COMPILE
```

Don't fall for this. You are not allowed to cosntruct a date or time object directly. Another trick is what happens when you pass invalid numbers to of(), For example:

```java
LocalDate.of(2018, Month.JANUARY, 32);    //throws DateTimeException
```

## Manipulating Dates and Times
Adding to a date is easy. The date and time classes are immutable .As you learned in Chapter 2 this means that we need to remember to assign the results of these methods to a reference variable so that they are not lost.
```java
LocalDate date = LocalDate.of(2018, Month.JANUART, 21);
System.out.println(date);                     // 2018-01-21
date = date.plusDays(2);                      
System.out.println(date);                     // 2018-01-23
date = date.plustWeeks(1);                  
System.out.println(date);                     // 2018-01-30
date = date.plusMonth(1);
System.out.println(date);                     // 2018-03-02
date = date.plusYears(2);
System.out.println(date);                     // 2020-03-02
```

The code is nice because it does just what it looks like. There are also nice and easy methods to go back in time. Such as minusDays(1), minustHours(10), minusSeconds(30) and so on... It is common for date and time methods to be chained.For example:

```java

LocalDate date = LocalDate.of(2020. Month.JANUARY, 20);
LocalTime time = LocalTime.of(5, 15);
LocalDateTime datetime = LocalDateTime.of(date, time).minusDays(1).minusHours(10).minusSeconds(30);
```

When you have a lot of manipulation to make, this chaining comes in handy. There are two ways that the exam creators can try to trick you. What do you think this prints?

```java
LocalDate date = LocalDate.of(2020, Month.JANURY, 20);
date.plusDays(10);
System.out.println(date);
```

It prints Janruary 20, 2020. Adding 10 days was useless because the program ignored the result. Whenever you see immutable objects types pay attention to make sure that the reurn value of a method call isn't ignored. The exam might also trick to see if you remeber that each of the date and time objects include.

```java
LocalDate date = LocalDAte.of(2020, Month.JANUARY, 20);
date = date.plusMinutes(10);              // DOES NOT COMPILE
```

LocalDate does not contain time. This means that you cannot add minutes. The following table shows the methods of LocalDate, LocalTime, LocalDateTime and ZOnedDateTime.

![Methods in LocalDate, LocalTime, LocalDateTime and ZonedDateTime](img/methodsLocalDateTimeZoned.png)

## Working with Periods
## Working with Durations
## Working with Instants
## Accounting for Daylight Savings Time

# Reviewing The String class

# Adding Internalization and Location
## Picking a Locale
## Using a Resource Bundle
## Creating a Property File Resource Bundle
## Creating a Java Class Resource Bundle
## Determining Which Resource Bundle to Use
## Formatting Numbers
### Format and Parse Numbers and Currency
## Fomratting Dates and Times
