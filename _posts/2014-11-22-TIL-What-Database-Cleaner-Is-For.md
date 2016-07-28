---
layout: post
title: TIL What Database Cleaner Is For
category: Coding
tags: Rails
year: 2014
month: 11
day: 22
summary: Each test is a transaction and will be rolled back at the end of the test. Database Cleaner is for tidying up left over data from integration tests where data can be committed since the database and 'browser' are running in different threads.
---

Ever seen [Database Cleaner](https://github.com/DatabaseCleaner/database_cleaner) in a project? "Strategies for cleaning databases in Ruby. Can be used to ensure a clean state for testing". I was playing around with the test helper and removed Database Cleaner to see what would happen. I noticed in a unit test that nothing is ever saved to the database. So what exactly does Database Cleaner clean? Let's try it out.

## Tests get rolled back
In a Rails 4.2 project with a user model and a basic test, create a user in the test. Now `rails c test` to have a look at your test database and try `User.count`. It should be 0 (make sure you don't have a fixtures file).

Why are there no users after the test has run? Turns out that Rails wraps each test in a database transaction. Everything gets rolled back at the end. We can test this out by committing the transaction with a `User.connection.execute("COMMIT")` at the end of the test. Now the `User.count` in the rails test console will be 1.

## Where does Database Cleaner come in?
So if each test is wrapped in a transaction, what on earth is Database Cleaner for? A bit of googling shows the [answer](https://github.com/jnicklas/capybara/blob/master/README.md#transactions-and-database-setup). Turns out that certain Javascript drivers with Capybara will kick off another thread. Transactions can’t see uncommitted changes from other transactions so Capybara will commit them to allow different threads to talk to each other. That's where Database Cleaner comes in; it'll tidy up the test database if you've got data left over from your tests.



