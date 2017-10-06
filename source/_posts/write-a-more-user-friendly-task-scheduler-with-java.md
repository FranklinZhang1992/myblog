---
title: How to write a user friendly Crontab like task scheduler with java
reward: true
date: 2017-10-06 17:43:39
tags:
  - Crontab
  - java
---

In previous version of the task scheduler (described [here](https://franklinzhang1992.github.io/2017/10/04/write-task-scheduler-with-java/)), you may find that the task is not executed at the time you expect. For example: If you create a task with a cron expression like `0 20 15 */3 *` at **2017-09-15 10:00**. Ideally, you expect it to be executed at today's 20:00 (**2017-09-15 20:00**). However, the truth is you will find that the next run time calculated by the Crontab algorithm is **2017-10-15 20:00:00**. You may feel strange because current time is 10:00 and there are still 10 hours left before it turns to 20:00, so why not the task will be executed at today's 20:00 and execute again at 20:00 three months later? Also, there is only one month between September and October, there seems to be no evidence proves that this is a every-3-months task.
This scenario is pretty common, especially for those who does not quite understand how Linux Crontab deals with the cron expression. So based on the algorithm described in my previous blog, here I will provide a solution to make the calculated execution time more user friendly.

###### Before we start, let's learn about how Linux Crontab deals with the cron expression and determine the next run time.

<!--more-->

Just like what we did in my previous blog, all fields of a cron expression can finally be converted to the form like `x-y/z` (x is start, y is stop and z is step) and based on the 3 values, we can generate an array which contains all valid values in this field. Let's use the expression `0 20 15 */3 *` as an example, for the month field the `*/3` can be converted to `1-31/3`, so we can conclude: start is 1, stop is 31 and step is 3. And with this 3 values, we can get an array `[1, 4, 7, 10]` (Because the first value in a month is 1). Still in this case, as current time is **2017-09-15**, so the nearest month we can get from the array is 10. Now you know why the next run time is October, right? Other fields keep the same logic so I just do not say much about them.

###### Do you understand the Linux Crontab's calculating next run time logic? If so, I am happy to say that you can go on reading my blog. Next, I will describe how I make the scheduler more user friendly.
## Goal
* The next run time should be calculated based on the current time. We still use the expression `0 20 15 */3 *` as an example, we want to calculated next run time should be as below:
  - If current time is **2017-09-15 10:00** (earlier than 20:00), then the next run time should just be today's 20:00 (**2017-09-15 20:00**).
  - If current time is **2017-09-15 21:00** (later than 20:00), then the next run time should be at 20:00 three months later (**2017-12-15 20:00**).
* For all every-X-minutes, hours etc. tasks, the interval between each execution times should always be the same. For example, if it is a every 7 months task, in original Linux Crontab, the task will be executed at January, August and next year's January etc. However, we can see that the interval between August and next year's January is 5 months, not 7 months. Obviously this will confuse some users. So we need to figure out a solution to let it be executed at next year's March and then the interval will still be 7 months.

## Solution
* The solution is simple, as you know, our algorithm of calculating next run time is just finding the nearest value from current time in the array list. So the secret of how we can make the scheduler more user friendly is just make the array list be a dynamic list, the each field's array should be updated when its parent time unit is nudged.
For example, if we nudge month field to next month, then the day-of-month field's array should be updated. We should count from the last valid value in the array list, plus the step, and use the new value as the start value of the new array, the step in the new array is just the same as the old one.
* Below is a demo of how we update the array. We use the expression `0 20 15 */7 *` as an example (we assume current time is **2017-09-15 16:00**), let's focus on the month field.
  - As current month is September, and the step is 7, in order to include September into the array, the minimum value in the array should be 2 and the array will be `[2, 9]`.
  - When we calculating the next run time, the nearest month is September, and the nearest date is 15th. As current time **16:00** is earlier than **20:00**, so the first run time should be **2017-09-15 20:00**. Goal 1 is satisfied.
  - When we continue to figure out the next run time, here comes the key, if we do not update the array, then the next run time will be in next year's February, but actually we want it to be executed at next year's April. So obviously the values in the month field's array are not accurate if we nudge to the next year.
  To resolve this, first, we need to pick out the last value in the array (**9**) and plus it with the step (**7**), we can use java class **Calendar** for the time calculation and we will easily get the new value which means next year's April (Note. in java Calendar, April is represented as 3 but in Crontab, April is represented as 4, so we need to do a conversion). Now let's use the new value as the first value in the new array and generate a new array with the step field, the new array will be like `[4, 11]`. With the new array, we can correctly get the next run time **2018-04-15 20:00**. Bingo.

###### Still, I will show you a demo to help you understand it.
Click [here](https://github.com/FranklinZhang1992/unity-learning/tree/master/java/TaskSchedulerAdvance) for the demo.

*****
转载请注明：[Franklin的博客](https://franklinzhang1992.github.io/)
