---
title: How to write a Crontab like task scheduler with java
reward: true
date: 2017-10-04 19:40:15
tags:
  - Crontab
  - java
---

Most people knows that there is a tool called Crontab in Linux and we can use it to execute some scheduled tasks. Normally, we just use it to call a script periodically and we write our business stuff in this script. This approach works for most of the cases. However, what if you want to integrate it to your project? You want to do some validations to let user know that they typed in a wrong cron expression and you want to store all tasks in your database instead of files. These will all be described in this blog.
*****
###### Before we start, a general introduction of Crontab time expression will be described (This is important because our self-written task scheduler will still based on the Crontab time expression):
## Crontab time expression
```
.---------------- minute (0 - 59)
|  .------------- hour (0 - 23)
|  |  .---------- day of month (1 - 31)
|  |  |  .------- month (1 - 12)
|  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7)
|  |  |  |  |
*  *  *  *  *
```
<!--more-->

1. The first column means minute, every-X-minutes can be represented as `*/X`, you can also specify some specified minutes with an expression like (each numbers are separated by a comma): `1,3,5` or a specified range with `1-3`.
2. The second column means hour, every-X-hours can be represented as  `*/X`, you can also specify some specified hours with an expression like (each numbers are separated by a comma): `1,3,5` or a specified range with `1-3`.
3. The third column means day, every-X-days can be represented as  `*/X`, you can also specify some specified days with an expression like (each numbers are separated by a comma): `1,3,5` or a specified range with `1-3`.
4. The forth column means month, every-X-months can be represented as  `*/X`, you can also specify some specified months with an expression like (each numbers are separated by a comma): `1,3,5` or a specified range with `1-3`.
5. The fifth column means day of week, every-X-days in a week can be represented as  `*/X`, you can also specify some specified days in a week with an expression like (each numbers are separated by a comma): `1,3,5` or a specified range with `1-3`.

## Some examples to help you understand Crontab time format better
1. `30 8 * * *` means the task will be executed at 8:30 every day.
2. `0 9 1,5,10 * *` means the task will be executed at 9:00 on the 1st, 5th and 10th day of the month.
3. `10 1 * * 0,6` means the task will be executed at 1:10 every Saturday and Sunday.
4. `8 8 1 */2 *` means the task will be executed at 8:08 on first day of every 2 months.
5. `30 8 10 3-10 *` means the task will be executed at 8:30 on 10th from March to October.

*****
###### Now you must know better about the Crontab time expression, it seems it is time to start write our own task scheduler based on the Crontab time expression.
## Validations
- As we know, each fields in the Crontab expression are separated by a white space, so the first step we need to do is to extract each field in the received string. We can simply do this with below code slice:
```java
String[] fields = cronString.split("\\s+");
```
- As each field has its own range, but they all have the same syntax, so we can first figure our current field's range and send its range to a common method for validation.
  - Minute: minimum value = 0, maximum value = 59
  - Hour: minimum value = 0, maximum value = 23
  - Day of Month: minimum value = 1, maximum value = 31
  - Month: minimum value = 1, maximum value = 12
  - Day of Week: minimum value = 0, maximum value = 7 (Because 0 or 7 both represent Sunday, so its range is 0-7, but to make our work easier, we need to convert 7 to 0 inside our code)
- In the common validation method, we only need three parameters, the string of the field, the minimum value allowed for this field and the maximum value allowed for this field. Now we have a question, the string of field can be like any of below forms, how can we make it easier to validate if the string is valid?
  - `*`
  - `1,3,5`
  - `1-5`
  - `*/2`
  - `1-5/2`
The answer is: we need to convert the `*` symbol to the range it represents (from the minimum value of this field to the maximum value), below is the code slice of how to do this:
```java
String convertedValue = fieldStr.replaceAll("^\\*", min + "-" + max);
```
- After the conversion, we just need to valid the string in below formats:
  - `1,3,5`
  - `1-5`
  - `1-5/2`
We can see that we can still meet an uncommon format `1,3,5`, we do not want to validate a string in this format separately, so what to do? An easy way is to separate the string by `,` symbol, then there will be only one type of the string.
  - `1-5` A range
  - `1-5/2` A range with steps
- After the separation, the converted string will contain below parts:
  - start: The start value of the field.
  - stop(optional): The stop value of the field, if there is only one specified value, then the stop value will just be the same as the start value.
  - step(optional): The step value of the field (AKA. every-X-minutes, hours etc.)
Here we may need to extract them from the string via regular expression `^(\\d+)(-(\\d+)(/(\\d+))?)?$` and below is what we will do after we match the string with the regular expression.
  - If the string does not match to the regular expression, then it means this field is not valid.
  - If the string matches to the regular expression, then:
    - Group 1 is the start value.
    - Group 2 is the stop value. If this value is null, then we need to set this value with the start value.
    - Group 3 is the step value. If this value is null, then we need to set this value as 1.
- Now it is time to validate if all of the three values are in the given range. The rules are as below:
  - min <= start <= max
  - min <= stop <= max
  - (stop - start + 1) >= step && step >= 1
- Since then, all validations are done. Next, we should make some preparations for calculating next run time. Actually, the only preparation we need is to put all valid values of the field into an array.
For example: If the month field is `*/3`, then the array will be like `[1, 4, 7, 10]`, all values in this array are calculated by the **start**, **stop** and **step**. See an example method of how to get the array.
```java
private List<Integer> getSteppedRange(final int start, final int stop, final int step) {
    List<Integer> steppedRange = new ArrayList<Integer>();
    if (start == stop && step == 1) {
        steppedRange.add(start);
        return steppedRange;
    }
    int len = stop - start;
    int num = len / step;
    for (int i = 0; i <= num; i++) {
        steppedRange.add(start + step * i);
    }
    return steppedRange;
}
```
- After we set the arrays for each of the field, all preparations for calculating next run time are done. Below is the work flow which can make the calculating algorithm easier to understand:
  - A table indicates the Crontab fields and the arrays which contain all valid values of the field.
```
+--------------+-----------------+
|    Field     |     Array       |
+==============+=================+
| minute       | minuteArray     |
+--------------+-----------------+
| hour         | hourArray       |
+--------------+-----------------+
| day of month | dayOfMonthArray |
+--------------+-----------------+
| month        | monthArray      |
+--------------+-----------------+
| day of week  | dayOfWeekArray  |
+--------------+-----------------+
```
  - The work flow of how we calculate next run time.
```
  +-------+
  | start |
  +-------+
      |   +--------------------------------------+
      +-->| find nearest month in the monthArray |
          +--------------------------------------+
                           |    +----------------------------------------------+
                           +--->| find nearest day of month in dayOfMonthArray |
                                | the day should also in dayOfWeekArray        |
                                +----------------------------------------------+
                                                     |     +------------------------------------+
                                                     +---->| find nearest hour in the hourArray |
                                                           +------------------------------------+
                                                                             |     +----------------------------------------+
                                                                             +---->| find nearest minute in the minuteArray |
                                                                                   +----------------------------------------+
```
- Now you have got a general idea of the algorithm of Crontab, so its time to have a look at my demo. This demo implements a simple task scheduler. You can copy the code, change the code and integrate it into your own project.
Click [here](https://github.com/FranklinZhang1992/unity-learning/tree/master/java/TaskScheduler) for the demo.

*****
转载请注明：[Franklin的博客](https://franklinzhang1992.github.io/)
