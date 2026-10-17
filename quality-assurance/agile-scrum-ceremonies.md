# Agile Scrum Team and Ceremonies — Two-Week Sprint Guide

Agile Scrum is a framework that helps teams deliver working software incrementally, gather feedback frequently, and continuously improve. In a typical Scrum project, the team works in fixed-length iterations called Sprints. A Sprint of two weeks (10 working days) is a common choice.

For interviews, especially at Senior SDET / QA Lead level, you should understand the Scrum roles, responsibilities, ceremonies, artifacts, and how Development and Quality Engineering collaborate throughout the Sprint.

## 1. Agile Scrum Team Structure

A Scrum Team consists of three accountabilities: Product Owner, Scrum Master, and Developers. In Scrum, the term Developers includes everyone who contributes to creating the product increment, including QA engineers, SDETs, automation engineers, and other technical specialists.

Product Owner (PO)

Owns product value and orders the Product Backlog

Scrum Team — Developers

Frontend • Backend • QA/SDET • DevOps and other skills needed to deliver the product

Scrum Master

Ensures Scrum is understood, coaches the team, and helps address impediments

Conceptual view: the Scrum Master is not a manager in a command hierarchy. The Scrum Team is self-managing and collaborates toward a shared goal.

### A. Product Owner (PO)

The Product Owner maximizes product value and is accountable for effective Product Backlog management.

Responsibilities:

- Defines and communicates the Product Goal.
- Works with stakeholders, customers, and business teams.
- Creates, clarifies, and orders backlog items.
- Ensures acceptance criteria are clear enough for the team to understand the expected behavior.
- Answers functional and business-related questions.
- Reviews feedback and adjusts backlog priorities.

Example: The PO prioritizes a user login enhancement ahead of a low-priority profile customization feature.

### B. Scrum Master

The Scrum Master is accountable for establishing Scrum and helping the team improve its effectiveness.

Responsibilities:

- Facilitates Scrum events when needed.
- Coaches the team on Scrum principles and practices.
- Helps remove or escalate impediments.
- Promotes collaboration between Product Owner, Developers, and stakeholders.
- Encourages continuous improvement.
- Helps protect the team from unnecessary disruptions without acting as a command-and-control manager.

Example: If the QA environment is unavailable and blocks Sprint testing, the Scrum Master helps the team resolve or escalate the issue.

### C. Developers — including QA/SDET

Developers are the cross-functional professionals who create a usable Increment each Sprint.

| Role / specialty                  | Typical responsibilities                                     |
| --------------------------------- | ------------------------------------------------------------ |
| Frontend Developer                | UI implementation, component tests, UI integration           |
| Backend Developer                 | APIs, business logic, database integration, unit tests       |
| Manual QA Engineer                | Exploratory, functional, integration, and regression testing |
| SDET / Automation Engineer        | UI/API automation, framework maintenance, CI quality gates   |
| DevOps / Platform Engineer        | CI/CD, infrastructure, deployments, environment reliability  |
| Security / Performance specialist | Helps validate relevant non-functional requirements          |

These are specialties, not separate Scrum accountabilities. One person may cover multiple specialties, and a Scrum Team does not necessarily need a dedicated person for every row.

Senior SDET interview point: Quality is the responsibility of the whole Scrum Team. QA is not merely a downstream team that tests after development is finished.

## 2. Agile Scrum Ceremonies / Events

Scrum formally defines five events, including the Sprint itself:

1. Sprint
2. Sprint Planning
3. Daily Scrum
4. Sprint Review
5. Sprint Retrospective

Backlog Refinement is an important ongoing activity, but it is not a formally defined Scrum event.

[Aguayo | Agencia de Publicidad, Estrategia CX y Digital](https://images.openai.com/static-rsc-4/CPsYwEUqAEvRM7GnhEcCrPSY6Owyj4-DTQpKHYjVsJxnAmU_aCPMteITY7GmRDQ3lEehLe-MOd_UpP7HSbB_IWJmVX5HercRXl70IoUtjOyrXhxx7Lfh1LWRJ22ObQE4Qj03praX5BcxgXdi2yXqnEzRu1t5p0kmCAyhfmxFc9s?purpose=inline)

1\. Sprint Planning

Day 1

Typical duration: up to 4 hours for a two-week Sprint; Scrum sets no fixed timebox for a two-week Sprint, but planning should be shorter when possible.

The team decides why the Sprint is valuable, what can be accomplished, and how the selected work will be done.

- Agree on the Sprint Goal.
- Select Product Backlog Items.
- Discuss acceptance criteria, dependencies, risks, and capacity.
- Create an initial implementation and testing plan.

QA/SDET contribution: Identify test scenarios, automation effort, test data, environments, dependencies, and risk areas.

[Scrum team participates in daily stand up on KANBAN board, mobile app developers](https://images.openai.com/static-rsc-4/RMAH3L04wCv5AceQ0V37OTGBMzqEXILqoWEr_VGSH162PlMssWsSJ7e8GKUGZ6B-oqMPGzBHZ5_AqZWC2md-aMyHV86256VjzA3MRYRrLRunpKztHa6bRIkFLnd1lsBV9lOS7NuTb-0OPV3Kqkx1WyD7I02d7G-Dk9y7BMx4eljgKEPEWozeMsAtjOdmJBUr?purpose=inline)

2\. Daily Scrum / Daily Stand-up

Every working day

Duration: 15 minutes

Developers inspect progress toward the Sprint Goal and adapt their plan for the next day.

Discuss:

- What progress have we made?
- What is the next step?
- What is blocking progress or putting the Sprint Goal at risk?

Many teams use the familiar three-question format, but Scrum does not require those exact questions.

QA/SDET contribution: Report test execution, automation status, defects affecting progress, environment issues, and any testing dependencies.

[スプリントプランニングとは - IT用語辞典 e-Words](https://images.openai.com/static-rsc-4/z4lKZJ7wSA1huI7y3Qj5T6xL_pHmTtalskmH4tNJyapcWF6VlvB6X6howzzOna-ynt6-2VSEJiTAxiFWW-A3cxb-iVG0P_E90VyAg03T422lBaAoYSegpXuH0bZZDWmgX9rT8PiffkkvAhHpSGiOXSVE_rQt5rsM8RMEY3u4ZwA?purpose=inline)

3\. Product Backlog Refinement

Ongoing

Typical duration: 30–60 minutes per session, depending on need; no prescribed Scrum timebox.

The team prepares upcoming work so it is sufficiently understood and ready for future Sprint Planning.

- Clarify requirements and acceptance criteria.
- Break large stories into smaller items.
- Identify dependencies and risks.
- Estimate effort or complexity.
- Identify positive, negative, boundary, API, UI, and non-functional test needs.

QA/SDET contribution: Review testability, identify missing scenarios, flag automation challenges, and expose specification gaps before implementation.

[Interactive Demonstration Examples: Engage Clients with Product Demos](https://images.openai.com/static-rsc-4/RZGoUg92qsMGGaKm1E18YB3od8G9VlztzRzRxGyqDHYPWyZn_QdmF5o8BaoVh4APXn807dZ7wYGexEj2RZOIl5lkYsHdPpC8plCd9rjdHPqE-i4T9A6kVZ7VEQTyOQvGKNHhsHwr8_eLaiWpKElQwP7QsK6j3TLc6wHJs7z49S8?purpose=inline)

4\. Sprint Review

End of Sprint

Typical duration: up to 2 hours for a two-week Sprint.

The team and stakeholders inspect the Sprint outcome and discuss what to do next.

- Demonstrate completed, usable functionality.
- Discuss the Sprint Goal and outcomes.
- Gather stakeholder feedback.
- Review market, business, and product changes.
- Adjust the Product Backlog as needed.

QA/SDET contribution: Demonstrate relevant quality evidence, automated test results, defects, coverage, and risks. Only work meeting the Definition of Done should be presented as part of the completed Increment.

[Sprint Retrospective: How Teams Get Better, Sprint by Sprint | by AkhilaKSaju | Medium](https://images.openai.com/static-rsc-4/auXZ5uTyHTuV5AOV1hIXdzTWrLpFR5QE_CYpxKCy40Nx8dzNlZGvSYC5n7kE08jr9mrrhZdqeydNoK8Fa6xwKRBD85T4r11CmmP36LDeclKZ5jaO-2_37xuQLDW4EKamw69FeR4AE5BfWfRGUe2osRQifiEvHGk4cBQDqf01nG4?purpose=inline)

5\. Sprint Retrospective

After Sprint Review

Typical duration: up to 1.5 hours for a two-week Sprint.

The Scrum Team examines how it worked and identifies improvements to quality and effectiveness.

Discuss:

- What went well?
- What did not go well?
- What should we change in the next Sprint?
- What action items can we commit to?

QA/SDET contribution: Discuss late testing, flaky automation, defect leakage, unstable environments, slow pipelines, and quality-gate failures. Agree on measurable improvement actions.

## 3. Two-Week Sprint — Day-by-Day Timeline

A two-week Sprint usually contains 10 working days. The following is an example schedule, not a mandatory Scrum calendar.

Day 1

Sprint Planning

Agree on Sprint Goal, select stories, plan development and testing.

Day 2

Development + QA preparation

Design, coding, test scenarios, test data and environment setup.

Day 3

Development + early testing

Unit tests, API checks, integration testing and automation.

Day 4

Backlog Refinement

Clarify and estimate upcoming stories; identify risks and dependencies.

Day 5

Integration and continuous testing

Merge work, run CI tests, review defects and Sprint progress.

Day 6

Feature completion + testing

Complete development, functional testing and automation.

Day 7

Regression and defect fixes

Execute regression, API/UI tests and resolve issues.

Day 8

Stabilization

Validate acceptance criteria, NFRs as applicable, and quality gates.

Day 9

Final verification

Complete eligible testing, assess release risks and prepare demo.

Day 10

Review + Retrospective

Demonstrate the Increment, gather feedback, and agree on improvements.

Daily Scrum occurs on each working day, including days with other Scrum events. Refinement can occur multiple times during the Sprint. Teams should not leave all testing until Days 8–10; testing and automation should start as soon as functionality becomes available.

### Approximate event time commitment

| Event                | Typical timebox for a 2-week Sprint |
| -------------------- | ----------------------------------- |
| Sprint Planning      | Up to 4 hours                       |
| Daily Scrum          | 15 minutes per working day          |
| Sprint Review        | Up to 2 hours                       |
| Sprint Retrospective | Up to 1.5 hours                     |
| Backlog Refinement   | As needed; no formal timebox        |

These are Scrum Guide maximums for the respective events where timeboxes are specified, not targets that every team must consume.

## 4. Scrum Artifacts and Commitments

Scrum has three artifacts, each with a commitment that helps provide transparency and focus.

| Artifact        | What it contains                                                  | Commitment         |
| --------------- | ----------------------------------------------------------------- | ------------------ |
| Product Backlog | Ordered list of work needed to improve the product                | Product Goal       |
| Sprint Backlog  | Sprint Goal, selected backlog items, and the plan to deliver them | Sprint Goal        |
| Increment       | Usable product functionality completed during the Sprint          | Definition of Done |

Example: For a login enhancement, the Sprint Backlog may include implementing login validation, API error handling, UI tests, and regression automation. The Increment is the integrated, usable functionality that meets the Definition of Done.

### What is the Definition of Done (DoD)?

The Definition of Done is the shared quality standard that work must satisfy to be considered part of the Increment.

A team's DoD may include:

- Code reviewed and merged.
- Unit and integration tests passed.
- Acceptance criteria validated.
- Required UI/API automation passed.
- Security, accessibility, and performance checks completed where applicable.
- No unresolved defects that violate the agreed DoD.
- Build and deployment checks passed where required.
- Documentation updated where necessary.

The Scrum Team must follow its Definition of Done. A story that is merely coded but has not met the DoD is not a completed Increment item.

## 5. How Development and QA Collaborate During a Sprint

### 1

Requirement & Acceptance Criteria

PO explains the expected business outcome.

### 2

Story Refinement & Test Design

Developers and QA identify scenarios, risks and dependencies.

### 3

Development & Unit Testing

Developers implement functionality and test their code.

### 4

Continuous Integration

Code is integrated; build, unit and automated checks execute.

### 5

Functional, API, UI & Regression Testing

QA/SDET validates the implementation and acceptance criteria.

### 6

Quality Gates & Definition of Done

The team verifies required quality standards and resolves failures.

### 7

Sprint Review & Feedback

Stakeholders inspect the usable Increment and provide feedback.

This workflow is illustrative rather than a mandatory sequential process. Development, testing, automation, and integration generally overlap throughout the Sprint.

For a Senior SDET, the key expectation is to shift testing left, integrate automated checks into CI, and identify quality issues before they become release blockers.

## 6. Common Scrum Interview Questions

Q1. What is the difference between Sprint Review and Sprint Retrospective?

Answer: Sprint Review focuses on the product, the Increment, and stakeholder feedback. Sprint Retrospective focuses on the team's process, collaboration, quality practices, and improvements.

Q2. Is QA responsible for the quality of the product?

Answer: Quality is the responsibility of the entire Scrum Team. QA/SDET contributes specialist testing expertise, automation, risk analysis, and quality visibility, while Developers also build quality through code reviews, unit tests, and integration tests.

Q3. Can the team add new stories in the middle of a Sprint?

Answer: The Developers and Product Owner can renegotiate the Sprint Backlog as more is learned, provided changes do not endanger the Sprint Goal. Scope can be clarified and adjusted collaboratively. If the Sprint Goal becomes obsolete, only the Product Owner can cancel the Sprint.

Q4. What happens if QA cannot finish testing before the Sprint ends?

Answer: If the work has not met the Definition of Done, it cannot be counted as part of the completed Increment. The team makes the unfinished work transparent, and any remaining work is reassessed and reordered in the Product Backlog. It is not automatically considered completed just because development has finished.

Q5. Who estimates story points?

Answer: The Developers who will perform the work are responsible for estimating it. QA/SDETs participate in estimation because testing, automation, integration, and regression effort are part of the total work.

Q6. What is the difference between a Sprint and a Release?

Answer: A Sprint is a fixed-length cycle for creating an Increment. A release is the delivery of product functionality to users or a production environment. A release may occur during a Sprint, at the end of a Sprint, or after multiple Sprints. Scrum does not require releases to wait until the Sprint ends.

Q7. What is the purpose of the Daily Scrum?

Answer: It is a 15-minute event for Developers to inspect progress toward the Sprint Goal and adapt their plan. It is not intended to be a status-reporting meeting to a manager.

## 7. Quick Revision

- Product Owner: What should we build next, and why?
- Scrum Master: How can we improve our use of Scrum and team effectiveness?
- Developers, including QA/SDET: How will we build and verify a usable Increment?
- Sprint Planning: What is our goal, what can we deliver, and how?
- Daily Scrum: Are we progressing toward the Sprint Goal?
- Backlog Refinement: Is upcoming work sufficiently understood?
- Sprint Review: What did we deliver, and what feedback did we receive?
- Sprint Retrospective: How can we work better next Sprint?
- Definition of Done: What quality standard must the Increment satisfy?

For interview preparation, remember this distinction: Scrum defines the framework and events; your team's engineering practices—such as CI/CD, automated regression, code review, and quality gates—help deliver a quality Increment within that framework.