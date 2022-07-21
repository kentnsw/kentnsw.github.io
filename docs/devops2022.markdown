---
layout: page
title: DevOps
permalink: /devops/
---

# DevOps Case Study

_-- Building A Cloud Governance Platform with Cloud Custodian_

Kent Ou / 2022-06   All rights reserved.

```
Preface - We Are 10 Years Late
Part 1: Enables Fast Flow
- Continuous Delivery
- A Peek at The Repo History
- PR Review: What Are We Reviewing?
- Automated Tests: A Cornerstone of Agile and DevOps
- Pay Technical Debts
- Continuous Deployment
- Feature Toggle: Globalize Policies in One Minute
- Right Solution: Where the 10x Improvement Comes From
- No Meeting Wednesday
- Fancy Documents That No one Read or Need
- Goal Alignment: Don’t Shoot The Messenger
- It Was Not My Business
Part 2: Enable Fast and Constant Feedback
Part 3: Continuous Improvement
```

## Preface - We Are 10 Years Late

DevOps principles and practices are compatible with Agile, with many observing that DevOps is a logical continuation of the Agile journey that started in 2001. If DevOps is the right trend, we are 10 years late.

This report is written by a frontline engineer who is working on building a cloud governance platform (the project) with an open-source tool Cloud Custodian. It’s for people who are interested in DevOps and what we can do differently from a DevOps perspective. This report, inspired by The DevOps Handbook, is composed of 3 parts corresponding to The Three Ways: Flow, Feedback, and Continuous Improvement.

<img src="/images/devops-3ways.png" alt="The Three Ways" width="366">

The project was started in September 2021 as a hobby project and got its goal re-defined as a cloud governance platform in January 2022. At the time of writing, it has been endorsed by the global engineering team including the APAC, AMER, and EMEA teams. It’s capable be extended to the operations, security, and architecture teams as well. Arguably it increased productivity by 10 times and changed the landscape of our governance tools. At the moment, the project has 30+ policies covering multiple areas and resources.

- Garbage collection, eg RDS
- Cost optimization, eg EC2
- Monitoring, eg cost
- Scheduler, eg nightly shutdown
- Security Compliance, eg CIS Benchmarks

## Part 1: Enables Fast Flow

This is about enabling a fast left-to-right flow of work from Development to Operations to the (internal) customer. Whatever we do, the goal is to increase the flow of the value stream.

### Continuous Delivery

The project requirements are naturally divided into small tasks. For example, garbage collect ElastiCache clusters, cost optimization for EC2. All developers are working in small batches on the develop branch, or working off the develop branch in short-lived feature branches that get merged to the develop branch regularly. Changes in the develop branch are regularly merged into the main branch by creating Pull Requests (PRs) for peer review. The main branch is always kept in a releasable state, and we can release on demand during normal business hours. This is how the project is doing continuous delivery.

```
-------------(main)---------------------/-----------/------------>
------\------(develop)-------/---------/-----------/----->
       \-----(feature)------/
```

The project has 226 PRs to the main branch so far. In another word, there are a total of 200+ releasable builds in the past 100 working days. Features or bug fixes do not have to wait days or weeks for the next release window. They’re ready for generating values at the time it is developed.

### A Peek at The Repo History

Below is a fragment of the repo history. The blue line is the main branch, the red line is the develop branch and the other lines are feature branches.

<img src="/images/repo-history.png" alt="Repo History" width="600">

As you can see, the Garbage Collect Elasticache and the Cost Optimize EC2 feature branches only lasted one day before they merged back to develop. Refactoring happened all the time on the develop branch, such as refined notes, DRYed (Don’t Repeat Yourself) code. Features are merged into main as a new deployable build once they are developed and tested.

### PR Review: What Are We Reviewing?

The main branch is prevented from any direct push. A pull request must be created and approved before changes can be merged into the main branch. A PR usually would take a couple of hours to a few days to get approved which doesn't work for this project that has 200+ PRs in the past 100 days.

How can the project maintain high quality without a peer review so far? The answer is that automated tests and self-discipline help a lot. To prove that, we have to dive deeper into the question of what we’re reviewing when we’re reviewing a PR.

- Review architecture? No, that usually won't be present at the code level.
- Review design, find blind spots and bad smells? Yes, that relies on developers' and reviewers’ experience. The reviewers should request a change to the PR to fix issues.
- Test it? Probably not. Not all reviewers have the knowledge, environment and/or time to do that. This is the most time-consuming, boring but yet important part of a change. It’s not just about the quality of the new feature it brings, it’s also about the quality of all the existing features that it could damage.

Self-discipline mitigates the risks of #2. This is a huge topic that can include a lot of things. Such as Keep It Simple and Stupid, long-term thinking. Here is a small example. It’s very attempting not to spend time writing a meaningful message when committing a change to the repo. But, each commit must mean something, and each commit message must tell everybody what is it about. We should never put something like “update code”, or “update dangerous_feature.py” into the history. In a long run, self-discipline is the best way to avoid trouble.

The automated tests can solve the issues of #3. Testing should be automated as much as possible. It must be self-service and available on-demand.

This does not suggest that peer review is dispensable. The peer review should be resumed once our review process is optimized for the team goal. Additionally, a review is not just about approval, but also about communication, improvement, and learning.

### Automated Tests: A Cornerstone of Agile and DevOps

Test is the only way to guarantee the source code on the trunk is releasable. Manual testing is not fast enough for continuous delivery, especially for people who are new to the project. They would not know what to do and how to test them. Automated testing, by far, is an effective way to ensure all the existing features are still working as expected after any changes have been introduced.

The project has 51 user acceptance test cases, each case executes the target policy a couple of times against different situations. They can be run locally at any time and finished within 3 minutes as self-service. The tests are triggered every time when a PR is created on Github. Those PRs will not be merged to the target branch if any test cases are failed.

A newcomer can develop new policies without spending time learning and testing old ones. The test cases act as a guard rail for new changes. Refactoring now can be done without fears and then technical debt can be paid.

In my opinion, automated testing is the most important practice in Agile/DevOps.

### Pay Technical Debts

We often find some bad smells in our system, and we want to refactor them but sometimes hesitated because of various reasons including “I am not familiar with the system”, or “the system is too critical to take the risk”, “the test and deploy process are too complex” and so on. As time goes by, the technical debts increase bit by bit, and eventually the whole system cracks.

The test cases of this project cover the most important scenarios to eliminate the above fears. For example, a test case of the Garbage Collect AWS RDS policy ensures a final snapshot is created before deleting the database instance(s). Refactoring happens all the time in the project to keep all policies in better shape.

An example of the contrary in this project is the delayed testing of the customized Cloud Custodian. The project customizes the Cloud Custodian for some bug fixes and feature enhancements. At the time I started to revise the code, a few automated tests were broken. That gave me a perfect excuse to skip the auto test step. I navigated changes very carefully. Yet 5 months later, after 72 careful and small changes, I have broken 32 test cases. When looking into some of them, I can’t remotely link them to the 72 suspects.

As Giray Ozil tweeted, “Ask a programmer to review ten lines of code, he’ll find ten issues. Ask him to do five hundred lines, and he’ll say it looks good.” Issues’ difficulty is not gently added together to become a big cute teddy bear, it multiplies together and turns into a horrible monster.
Continuous Deployment
In this project, every change to the main branch goes to production automatically.

A Jenkins pipeline monitors the branch hourly and applies the latest version to the production environment. The pipeline takes care of the deployment of policies and infrastructures, e.g. creating AWS SQS queue, deploying and updating mailer (lambda), deploy policies (lambda, Config rules). No other teams or resources are required during this process.

This enables us to local the root cause much quicker and easier when issues are reported. Most of the time, it is one of the changes introduced in the last deployment.

### Feature Toggle: Globalize Policies in One Minute

Ideally, policies are applied to multiple BUs. On one hand, all BUs resources should be governed by the same standards. On the other hand, teams in different BUs should join the effort and share the artifacts they produce. That’s the beginning of organizational learning.

Sometimes we do not want such a policy deployed to all BUs at the same time. Thus we need to decouple the delivery and the deployment by using the feature toggle.

The project has a deployment configuration file for each BU to serve as a feature toggle. Developers can submit policies at any time, while the BU admin decides when the policy is to be applied to what BU accounts.

That being said, with the support of underlying infrastructure including an account list and a role in each account, policies can be globalized by a ‘tick’.

### Right Solution: Where the 10x Improvement Comes From

Any company that heavily uses public cloud technologies must have governance requirements. From this point of view, they are more alike than different. There must be some mature solutions on the market already as it’s such a huge market and so many years have passed. Acquiring a mutual solution should be the first choice. After all, a home-grown solution is the last resolution in the software business.

Cloud Custodian is an open-source solution for that. It significantly improves the overall productivity compared to a home-grow solution, although it can’t compare to those battery-included SaaS ones.

To complete a garbage collection functionality like GC RDS, it takes around 200 lines of code plus 150 lines of automated test code. For a similar functionality in a home-grow solution, it takes 5000 lines of code plus 500 lines of automated test code. That’s the 10 times improvement begins.

Because of the project, we are having a single centralized repo for most of our governance rules rather than dozens of repos and pipelines.

### No Meeting Wednesday

A company observed that there were too many meetings in the leaders' calendar resulting in no time for doing any productive work. Thus a No Meeting Wednesday policy was introduced, and finally, the leaders can have one full day to get the ball rolling.

Findings generated by the project were routed to ServiceNow. Some teams feedback that they need those tickets on Jira because the ServiceNow is for a different purpose. So the CloudOps team copied the tickets to Jira for them until the Jira integration was available in May. It sounds like the problem was solved, but I’d say a new problem has begun. Data meant to be together now got divided into two systems. Any work that required them now has to communicate with two systems.

To my observation, the leaders got more meetings on the other four days and had to utilize office corners and canteen because all meeting rooms were booked out. It was the ways we work, the teams we organized, and the processes we created that require more communications, more coordination, and more meetings. We don’t blame Wednesdays.

### Fancy Documents That No one Read or Need

There was a well-structured and elaborated document. At the time of reading, everything was accurate. However, as one of the target audiences, there were only two sentences in that document that matter to me.

On the Confluence site, pages created two years ago that have only two unique readers are not hard to find. Pages that have more than 10 readers could be popular ones. Pages that have more than 30 readers, perhaps the author could call himself/herself the number one (writer).

This project has one main page and 3 sub-pages that have 16 readers in total. The pages only contain information for non-technical people, such as a policy list, and some how-to steps. Other information is softly documented in the repo, together with the code. From some point of view, it is not well documented. But I don’t feel guilty about that.

Fewer documents need less time to create and maintain. Soft documented and near the code make them executable or easier to sync with the code. The last thing we want is the document saying one thing and yet the system doing another.

### Goal Alignment: Don’t Shoot The Messenger

Besides automatic remediation, the project has been generating 300+ tickets for the business to review or take action. Thousands of cloud assets/resources have been monitored, cleaned up, or cost-optimized.

Let’s imagine that the business owner is a captain who leads a team to fight a life or death battle, and millions of dollars of resources are made ready to ensure the team has the necessary support to win it. Yet here comes a door knocking and a messenger, “Dear Captain, you have a 9.5 dollars cost optimization opportunity to investigate. Please take action in 14 days”. No doubt, that messenger will get shot one day.

But, this is not the messenger’s fault because he/her has not been told what are the business team’s goals, plans or right timings. The messenger might have walked a long way to make to the camp, and sometimes he/she does carry some very important messages.

Often these findings are found too late because the workloads have gone live, or damages have been done. For example, we found $30,000 a month cost savings recently, which from one point of view, that’s great. But from another point of view, it would be even better if we can find them earlier.

But, that is not the messenger’s business, isn’t it?

### It Was Not My Business

The “Wall of Confusion” is a DevOps term popularized by Andrew Clay Schafer (AgileRoots 2009 ~17:00 mark) and Lee Thompson (Dev2Ops Interview). It refers to the phenomena where one group in a value stream approaches their job as complete when they’ve passed it onto the next group.

<img src="/images/the-wall-of-confusion.png" alt="The Wall of Confusion" width="274">

Once a time, I asked a governance rule author a question: how will the rules be implemented? The author said it is out of scope. I agree with that according to the author’s responsibility. But, I don’t clearly see if it is in anyone’s scope. Without seeing it from an end-to-end perspective, I guess no one could answer

- Who are the stakeholders and are they aware of this?
- Who’s going to implement the rules?
- Have the rules been implemented? When?
- How many percentages of assets comply with the rules?
- Any practical issues related to the rules?
- Whom to notify if the rules are changed?

Let's back to the project and imagine how an author, a developer and other stakeholders can work together, on the same repo and the same PR.

An author adds the following rule in garbage_collection.md, files a PR and requests related stakeholders to review and/or be notified.

```
AWS EBS Governance Rules
    - EBS snapshots that are older than 1 year, un-preserved and unused should be deleted
```

A developer picks up the task and implements it in gc_ebs_snapshots.yml, submits it to the PR along with an automated test case test_gc_ebs_snashots.py, and requests more stakeholders (if needed) to review the changes.

```yaml
policies:
  - name: gc_ebs_snapshot_delete
    resource: ebs-snapshot
    description: older than 1 year, un-preserved and unused should be deleted
    filters:
      - "tag:Preserve": absent
      - type: age
        op: gt
        days: 365 # need to update test case as well when it changes
      # unused means snapshot is not used by launch-template, launch-config, or ami.
      - type: unused
        value: true
    actions:
      - type: delete
```

```python
def test_gcebssnapshot_basecase(aws_credentials):
  pt = Ec2PolicyTest()
  volIds = pt.create_volumes(pt.requiredTags)
  sid = pt.client.create_snapshot(VolumeId=volIds[0])["SnapshotId"]

  with freeze_time(nDaysLater(365)):
      pt.run_policy(policy)
  assert sid not in [r["SnapshotId"] for r in pt.client.describe_snapshots()["Snapshots"]]
```

Reviewers post their comments on the PR and the author and developer update the rule and implementation to solve issues. Finally, the PR got approved. Once the PR is merged to the trunk, it will be deployed to the production environment automatically in a few minutes. Then, people move on to the next happy cycle.

## Part 2: Enable Fast and Constant Feedback

## Part 3: Continuous Improvement
