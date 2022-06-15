---
title: Write Your personal money manager for fun and free !!!
slug: writing-personal-money-manager-for-fun
date_published: 2020-09-11T12:41:42.000Z
date_updated: 2020-11-29T07:05:49.000Z
tags: python, sideproject, gmail, googledrive, googlesheet, personalfinance, hack
---

At some point in time you may have used a money manager to track your expenses, categorize them etc. All these work great but the only issue is that you need to share *very *sensitive information with a third party. If they are tracking SMS, then they know all about the salary, bank credit , debit etc and I don't really feel comfortable with that.

For every expanse whether it is through credit card/ debit card/ net banking / NEFT/ UPI etc, bank always sends an email providing details about the transaction. Why not use these email to track all the expanses without sharing it with any third party.

Here is a basic setup to do this. I'm using gmail and other google services to build this app since google already knows all about my emails and I don't want to send any thing outside of it .

The basic idea is - 

1. Write an app which can connect to your gmail and reads mails for a given date range and from a sender. This sender is your bank's email address which sends you alert for each transaction.
2. Parse those email and extract information about transaction date, amount, seller etc .
3. Store this information in google sheet. Each month's data goes in a separate sheet.
4. Read this data from corresponding sheet and generate email message providing you with expense summary.
5. Run this app at the a regular interval to get the email summary as well as update sheet. You can run it on your system, if you have a system where you could schedule it or on raspberry pi etc.

This is available at [https://github.com/kapilgarg/passbook](https://github.com/kapilgarg/passbook)
