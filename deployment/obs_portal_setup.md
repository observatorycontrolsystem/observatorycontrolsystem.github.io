---
layout: page
title: Observation Portal Initial Setup
---

# Observation Portal Initial Setup

There are a things that must be added to a newly deployed Observation Portal to begin accepting Observation Requests. You should already have a superuser account created which you can use to log in to the admin interface of the Observation Portal. All steps are accomplished by clicking on the corresponding model in the `/admin/` endpoint and clicking the "Add a *" button in the upper right corner of that model's page.

A prerequisite to using the Observation Portal is having the Configuration Database setup. If you have not yet set up your Configuration Database, stop this guide and first follow the instructions in the [ConfigDB Setup]({% link deployment/configdb_setup.md %}) section.

## Setup a Proposal

The first step to accepting Observation Requests is to set up a proposal with time allocated to it for the **Instrument Types** you created in the Configuration Database.

1. Add a Semester: This represents an Observing Semester at an Observatory, which is a period of time during which time on instruments is allocated and available for use by proposals. These should be non-overlapping, but you can create as many of these as you want, i.e. yearly, bi-yearly, or just have a single one with a time range of all time.

2. Add a Proposal: Members of a proposal, which include the proposal's Principle Investigator (PI) or Co-Investigator(s) (CoI), are allowed to submit Observation Requests for instruments types that their proposal has been allocated time on for a particular observing semester. At a minimum, you must specify the **Id** of the proposal, which will be used when submitting Requests on that proposal, and then the **tac priority**, which is used by the Adaptive Scheduler when it is calculating the observing schedule. The direct submission field should be checked if you want to allow members of the proposal to to create Observations directly using the Observation Portal `/api/schedule/` API - otherwise members of the Proposal will only be allowed to submit Observation Requests. On the bottom of the **Add a Proposal** screen, click on **Add a Time Allocation** to allocate time to this Proposal. You must initially specify the Standard (std), Rapid Response (rr) and Time Critical (tc) allocation of time in hours, then select the Observing **Semester** this Time Allocation will apply to, and finally select one or more **Instrument Types**. The **IPP limit** and **IPP time available** will auto-fill to 10% and 5% of the **Standard Allocation** value if you leave them blank.

3. Add a Proposal Membership: Each Proposal should have at least one PI, who will be able manage the proposal's COIs, including adding and removing COIs and setting limits on how much of the proposal's allocated observing time they can use. To add this initial PI, select the **proposal** and **user** by ID using the search button, and then select a **role** of Principle Investigator. The **time limit** field is used to limit the specific user to a specific amount of time they are allowed to use on that proposal, with the defalt of -1 being no limit.

All users that are members should now by able to submit Observing Requests specifying this newly created proposal. New user accounts can be requested by users by using the Observation Portal's registration form, and then activated via that user's email. Those accounts will then need to be added onto a proposal to be able to submit Observation Requests and see the proposals data. A PI can also invite new members to join their proposal even if that user does not yet have an account in the Observation Portal - that user will receive an email letting they know that they have been invited to join, and that they can register an account.
