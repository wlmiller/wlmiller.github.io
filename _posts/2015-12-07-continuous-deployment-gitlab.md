---
layout: post
title:  "Setting up continuous deployment in  GiLab for a Flask project"
date:   2015-12-07 17:00:00
categories: python nose-tests continuous-integration continuous-deployment
author: Neal Miller
---

Recently, I've been working on a Python [Flask](http://flask.pocoo.org/)-based web application.
We use [GitLab](https://about.gitlab.com/) as our source control system as RigMinder, and an important part of getting the project set up was having an automated process for testing and deployment.
After a bit of setup, it works very well; I thought I'd post a quick description of how we got it set up.

## Installing GitLab Runners
In order to run GitLab continuous integreation, a [GitLab Runner](https://gitlab.com/gitlab-org/gitlab-ci-multi-runner) is required.
GitLab runners run the tests provided and send the results to GitLab CI.
Installing and setting up a runner is easily; simply run the binary from GitLab with the `register` command.

```
sudo gitlab-runner register
```

Follow the instructions, providing the registration token, which can be found in GitLab by navigation to your project, then to Settings > Runners.

## .gitlab-ci.yml
`.gitlab-ci.yml` is the configuration file used by the runner to execute continuous integration.
It is placed in the root directory of the project.
Because I wanted continuous *deployment*, my `.gitlab-ci.yml` consists of two stages:

{% highlight yaml %}
stages:
    - test
    - deploy
    
test:
    stage: test
    script:
        - scp app/tests/*_test.php root@{test server}:/var/www/
        - nosetests --with-coverage --cover-package=app --cover-branches
        - ssh root@{test server} "rm /var/www/*_test.php"
    only:
        - master
        
deploy:
    stage: deploy
    script:
        - rm -rf /var/www/html/portal
        - mkdir /var/www/html/portal
        - cp -r * /var/www/html/portal
{% endhighlight %}

The `test` stage performs three actions:

1. Copy some necessary supporting test files to the test server.
2. Run the included tests (using the [nosetests](https://nose.readthedocs.org/en/latest/) library).
3. Clean up the copied test files.

Assuming the `test` stage passes, the `deploy` stage runs, which simply replaces the existing app directory with the new version.

## Extracting coverage
Note that above I included the `--with-coverage` flag for nosetests.  
This provides an output similar to that shown below, from which the total test coverage can be extracted.

{% highlight python %}
Name                           Stmts   Miss Branch BrPart  Cover   Missing
--------------------------------------------------------------------------
...  
app/users.py                       0      0      0      0   100%   
app/users/controllers.py         309     22    116      7    93%   26-27, 124, 167, 170-172, 251, 262, 271, 281-282, 286, 295-299, 375, 395-396, 408-409, 25->26, 90->93, 123->124, 166->167, 250->251, 261->262, 270->271
app/users/models.py              361     27    166     15    92%   89, 107-108, 112, 185, 197-198, 209, 218-220, 376-378, 413-415, 443, 446, 453-455, 459, 468, 483, 567, 569, 88->89, 111->112, 155->157, 175->177, 184->185, 207->209, 247->252, 249->252, 442->443, 445->446, 458->459, 467->468, 477->483, 566->567, 568->569
...
--------------------------------------------------------------------------
TOTAL                           1943    210    688     68    87%   
----------------------------------------------------------------------
Ran 114 tests in 294.046s

OK
{% endhighlight %}

That last number -- `87%` -- is the total coverage number I'm after.
By navigating with GitLab to your project > Settings > CI Settings and placing the followin regular expression in the "Test coverage parsing" field.

{% highlight python %}
\d+\%\s+$
{% endhighlight %}

This pattern matches a sequence of digits, followed by the `%` symbol, followed by one or more whitespace characters, followed by the end of a line.
The result is that GitLab is able to extract the `87%` coverage metrics, which is displayed alongside the commits in GitLab.