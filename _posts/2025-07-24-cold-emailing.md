---
layout: post
title:  "Cold Emailing in Big Tech"
date:   2025-09-05 12:52:08 -0800
categories: general
---
Many new grads and entry level software engineers are struggling to land jobs. I too was struggling until I came accross cold emails. Cold emailing is essentially just mass emailing recruiters. In my experience sending cold emails to recruiters in addition to just applying increased the number of online assessments and interviews I was getting.

To get started with cold emailing you need an email template. Following is the exact template I am currently using

## Email Templates
""""

Hi $RECRUITER,

I came across your profile on LinkedIn and understand you are recruiting for $COMPANY_NAME. My name is Rahul, and I applied to $COMPANY_NAME for a software engineering position. I have been working at $CURRENT_COMPANY_NAME$ as a Backend Software Engineer since January 2024.


I was wondering if there was any way to schedule a phone interview or online assessment with $COMPANY_NAME as I am interested in the opportunity. I'm ready to interview ASAP if it is possible. I have attached the resume I applied with.


I am really interested in $COMPANY_NAME and I'd love to learn about your experience, exciting projects you're working on, and potential career opportunities where I could add value to your teams. Would you be open to a 20 minute virtual coffee chat over the next couple of weeks, if it's not a good time, I completely understand. Either way, hope you have a great week. 

Cheers,
Rahul

""""


You also want a catchy subject so the recruiter actually clicks on the email. You need to have good clickbait email subject. My email suject was **CURRENT_COMPANY_NAME Software Engineer from Georgia Tech interested in $recruiter_company**.


It's important to keep the emails short and to the point. Recruiters are busy people and don't have time to read through all your work experience, that is what your resume is for. The goal of sending the email is to expedite the interview process and potentially talk to a recruiter.

# Finding Recruiters and email addresses

The hardest part of the cold emailing process is finding the recruiters and their emails. How I went about doing this is using LinkedIn to find technical recruiters at the company. My search query would be something like "Technical Recruiter $COMPANY_NAME". To find the email I used hunter.io, a website which has a database email addresses for recruiters and sales people. Keep in mind hunter.io has a free plan where you can have 25 searches free for the month but more than that you will have to upgrade to the paid plan which is quite expensive ($50/month). There are alternatives to hunter but I have not tried them. The way I look at it spending $50 for a few months is well worth it if it gives you an edge to land a potential $150k+/year job. 

# When to send Emails
I usually apply to a position and then send at minimum 3 emails to recruiters at the company. I apply in the evenings or at night but I schedule the emails to send during the workday. I send emails between 10:00 AM-11:00 AM and 1:00 PM-2:00 PM. I find these times ideal as they will get to the top of a recruiters inbox during the workday. 10:00 AM is late enough where morning meetings are wrapping up and it is kind of a lull period before lunch break. I skip the afternoon time on Friday because Friday afternoon is basically the weekend.

# Streak
You can use gmails built in schedule sending capabilities to schedule the emails but I use [Streak CRM](https://www.streak.com/). This is a chrome extension that adds features on top of Gmail such as read receipts and schedule sending. The read receipts are especially useful because you know if a recruiter is looking at your emails and can choose specifically to send followups to the recruiters that are opening your emails.

# Automating the process
When I started doing this it was extremely manual where I had to first find a recruiter then find the email. Then I would need to copy and paste the email template from google docs into the gmail and edit all the variables such as the company name and recruiter name and subject line and also attach my resume. Missing to change any of these variables is a little embarassing but you might have also blown up your shot because of a silly error on your part. I have sent wrong company name, wrong recruiter name, or forgotten to attach resume more times than I would like to admit

![email fumble]({{ "/images/email-automater/fumble.png" | absolute_url }})

To solve these problems I created a small Python tool that automates the process of creating the emails. It is on my Github called [email-automater](https://github.com/rvaidun/email-automater). By using the tool you can send emails straight from your command line. See README for more instructions on usage.

# Wrap it up

One thing to keep in mind is to be persistant about it. A lot of times the recruiters will ignore you and never get back to you. One day I scheduled 50 emails and had maybe 3 responses with 2 of them being automatic out of office responses. It's easy to get discouraged but keep going and you will get some good responses back. Also make sure you practice leetcode and keep your DSA skills sharp. You don't want to fumble the interview once you get the interview. I would just go through neetcode top 100 and also grokking the coding interview and have that on lock.

Hope this helps some of you get started with cold emailing. Below I have pasted a snippet of a successful email thread with a recruiter to give you all hope that it is possible to land the job and interview, Just be persistant

![email success]({{ "/images/email-automater/success.png" | absolute_url }})
