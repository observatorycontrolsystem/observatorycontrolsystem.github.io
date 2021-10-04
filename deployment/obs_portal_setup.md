---
layout: page
title: Observation Portal Setup
---

# Observation Portal Initial Setup

There are a things that must be added to a newly deployed Observation Portal to begin accepting Observation Requests. You should already have a superuser account created which you can use to login to the /admin interface of the Observation Portal. All steps are accomplished by clicking on the corresponding model in the /admin interface and clicking the "Add a *" button in the upper right corner.


## Setup a Proposal

The first step to accepting Requests is to setup a proposal with time on it for the **Instrument Types** you created in the Configuration Database.

1. Add a Semester: This represents an Observing Semester at an Observatory, which is a period of time during which time on instruments is allocated and available for use for proposals. These should be non-overlapping, but you can create as many of these as you want, i.e. yearly, bi-yearly, or just have a single one with a time range of all time.

2. Add a Proposal: This can have time allocated to it on instrument types during observing semesters, which is available for use by any Principle Investigator (PI) or Co-Investigator (COI) on the proposal. At a minimum, you must specify the **Id** of the proposal, which will be used when submitting Requests on that proposal, and then the **tac priority**, which is used by the scheduler when evaluating where to place Observations. The **direct submission** field should be checked if you want to be able to submit Observations directly with this Proposal through the Observation Portal /api/schedule/ API - otherwise the Proposal will only be allowed to submit Requests. On the bottom of the **Add a Proposal** screen, click on **Add a Time Allocation** to allocate time to this Proposal. You must initially specify the Standard (std), Rapid Response (rr) and Time Critical (tc) allocation of time in hours, then select the Obsering **Semester** this will apply to, and select one or more **Instrument Types**. The **IPP limit** and **IPP time available** will auto-fill to 10% and 5% of the **Standard Allocation** value if you leave them blank.

3. Add a Proposal Membership: Each Proposal should have at least one PI, who will be able to control the time allocation and access of COIs on the proposal under them. To add this initial PI, select the **proposal** and **user** by Id using the search button, and then select a **role** of Principle Investigator. The **time limit** field is used to limit the specific user to a specific amount of time on that proposal, with the defalt of -1 being no limit.

All users that are members should now by able to submit Observing Requests specifying this newly created proposal. New user accounts can be requested by users hitting the site, and the superuser/administrator can approve those accounts. Those accounts will then need to be added onto a proposal to be able to submit requests and see the proposals data.
