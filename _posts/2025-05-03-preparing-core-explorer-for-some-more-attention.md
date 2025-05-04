---
layout: post
title: 'Preparing Core Explorer for some more attention

  '
date: 2025-05-03 13:31 -0500
---


## Current Status

We've noticed there's a lot of attention about this [`OP_RETURN` discussion](https://github.com/bitcoin/bitcoin/pull/32359). And we'd like to leverage that for our own interests, getting attention and perhaps collaborators for [Core Explorer](https://github.com/coreexplorer-org).

So far we've tried a few different approaches.

## Today's effort

Since we're working with this new PR, it's going to be helpful to get some updated data. So we'll go over the first version of Core Explorer, create a new github API key, and plug that into the scripts which scrape data from the repo... then we'll get some tacos while that runs.

### Setup a github api key for scrapin'

API docs for that: 
https://docs.github.com/en/rest/authentication/authenticating-to-the-rest-api?apiVersion=2022-11-28#basic-authentication

there was an old github token, but it seems not to be valid any longer... so we'll make a new one that's valid for 30 days.

okay, we had an old token, but let's make a new one in `.zprofile`


`export THIRTY_DAY_GITHUB_SCRAPE_TOKEN="ghp_blahblahblahblah"` * note that ths was originally `30_DAY_TOKEN` but environment memory needs to start with a letter, not a number... so things got broken. 


Alright, so, in order to get the functions to do the thing, we've got a script in repo_explorer called `repo_explorer/github_scrape_commits_or_pulls.rb` and you can change the value of 

we got the new API code, put it in our repo & then used the two scripts to scrape


  `GITHUB_API_TOKEN=$THIRTY_DAY_GITHUB_SCRAPE_TOKEN  ruby github_scrape_commits_or_pulls.rb bitcoin/bitcoin all_commits_before_may_2025.csv`
  `GITHUB_REPO_TOKEN=$THIRTY_DAY_GITHUB_SCRAPE_TOKEN  ruby github_prs.rb bitcoin/bitcoin all_commits_before_may_2025.csv `
 
note the subtle difference between the two `GITHUB_REPO_TOKEN` and `GITHUB_API_TOKEN`. that's a thing that can get resolved with the 2nd script which is `scrape_commits_or_pulls`. There's some duplicated code that can get consolidated. I think even in writing these notes, i've actually got the commands wrong, go figure. anyway, figure it out. and the more verbosely named script can get fixed to cover both the commits, and the prs. and potentially onter data types that need to be accounted for from Github's centralized system (specifically it may be smart to get the individual commits in order to get the message history & "NACK" "ACK" related things from discussions (preferably soon before it gets deleted forever, accidentally or otherwise).


