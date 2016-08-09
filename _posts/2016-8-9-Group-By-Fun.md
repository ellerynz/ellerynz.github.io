---
layout: post
title: Group by fun
category: Rails
tags: Rails relation group
year: 2016
month: 8
day: 9
summary: ActiveRecord's group method can take an array which changes the output
---

Noticed a subtle difference in the output from a `group` on a relation in rails.
Let's say we want to count the number of users by the month they were created in.
Here's two ways to write that query:

```
# 1
User.group("YEAR(created_at), MONTH(created_at)").count
(0.5ms)  SELECT COUNT(*) AS count_all, YEAR(created_at), MONTH(created_at) AS year_created_at_month_created_at FROM `users`  WHERE `users`.`deleted_at` IS NULL GROUP BY YEAR(created_at), MONTH(created_at)
=> {12=>8, 1=>1}

# 2
User.group("YEAR(created_at)").group("MONTH(created_at)").count
(0.5ms)  SELECT COUNT(*) AS count_all, YEAR(created_at) AS year_created_at, MONTH(created_at) AS month_created_at FROM `users`  WHERE `users`.`deleted_at` IS NULL GROUP BY YEAR(created_at), MONTH(created_at)
=> {[2015, 12]=>8, [2016, 1]=>1}
```

The second one was what I was after. Turns out `group` provides a different name for each element in the array.
So the chaining above could have been written as: `User.group("YEAR(created_at)", "MONTH(created_at)").count`.
http://api.rubyonrails.org/classes/ActiveRecord/QueryMethods.html#method-i-group
http://api.rubyonrails.org/classes/ActiveRecord/Calculations.html#method-i-count


Postgres equivalents

```
User.group("EXTRACT(year from created_at), EXTRACT(month from created_at)").count
User.group("EXTRACT(year from created_at)", "EXTRACT(month from created_at)").count
```

