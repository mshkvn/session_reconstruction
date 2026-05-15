# Session Reconstruction & Friction Analysis in Exams with Proctoring

## Overview
This project reconstructs user exam attempts from raw event logs in an online proctoring system and identifies technical bottlenecks during the onboarding process before an exam starts.

The main challenge was reconstructing real user sessions from noisy event streams without a reliable session identifier.

The resulting pipeline allows:
reconstruction of successful and failed exam attempts,
detection of onboarding friction,
identification of массовых technical issues,
proactive monitoring of client-specific problems.

## Business Context
Before starting an online proctored exam, users must complete a technical onboarding flow that may include:
- browser permission checks,
- webcam and microphone validation,
- network testing,
- identity verification (if required),
- document upload (if required).

This stage represents the highest level of interaction between the user and the proctoring platform. Once the exam begins, most interactions occur inside the testing platform itself and proctoring system becomes almost invisible.

At the same time, technical problems during onboarding directly affect exam completion rates and user experience.

## Problem
Initially, information about onboarding failures was available only through support tickets submitted by users. This created several limitations:
- no visibility into the scale of technical issues,
- inability to detect problems proactively,
- no reliable way to compare onboarding quality across clients,
- no structured dataset of failed onboarding attempts.

The main technical challenge was that the raw event logs did not contain a reliable session identifier. The logs consisted of a continuous stream of:
- user events,
- system events,
- partially incomplete records,
- missing exam termination events.

As a result, user attempts had to be reconstructed heuristically from noisy event sequences.

## Objective
The project aimed to:
1. Reconstruct user journeys from login to exam start using raw logs.
2. Identify key onboarding bottlenecks and technical failures.
3. Build a dataset of successful and failed attempts for further analysis.
4. Enable proactive monitoring of large-scale onboarding issues.

## Data Challenges
The system stored several independent data sources:
- exam session data,
- user metadata,
- raw application logs.

However:
- exam session tables only contained successfully launched exams, failed onboarding attempts were absent,
- raw logs had no unified session identifier,
- some terminal events (_stoppedAt_) were missing user information,
- logs contained substantial system noise.

The project therefore focused on event stream reconstruction and heuristic sessionization.

## Methodology
### Attempt definition
An attempt was reconstructed from user actions and defined as:
- starting from a login event,
- ending either:
  - with exam termination (_stoppedAt_),
  - or after 15+ minutes of inactivity without exam start,
  - or after exam start followed by 60+ minutes of inactivity when termination events were missing.

An attempt was considered successful if it contained a _startedAt_ event.

## Session Reconstruction Pipeline
The reconstruction pipeline included:
- Recovery of missing user identifiers for some stoppedAt events using the shared record field.
- Filtering only candidate-related events.
- Calculation of inter-event time gaps.
- Detection of inactivity-based session boundaries.
- Reconstruction of attempts using heuristic rules.

Noise filtering:
- removal of attempts containing only 1–2 events,
- exclusion of empty logins without meaningful interaction.
- Aggregation of session-level metrics.

## Key metrics
The reconstructed dataset included:
- _attempt_success_ — whether the exam was successfully launched,
- _attempt_id_ — sequential attempt number per user,
- _attempt_size_ — number of events inside the attempt,
- _equipment_check_time_min_ — onboarding duration,
- _amount_of_errors_ — number of technical errors,
- user metadata:
  - device,
  - browser,
  - location.

## Validation
The reconstruction quality was validated in two ways:
The number of reconstructed successful attempts closely matched independently stored exam session counts.
Samples of failed attempts were manually reviewed to validate session boundaries and reconstruction quality.

The observed validation error was below 0.1% of sessions.

## Results
The project produced:
- a reconstructed session-level dataset from raw event logs,
- identification of failed onboarding attempts,
- monitoring of onboarding success rates,
- automated detection of frequent technical problems.

The analysis established a practical onboarding baseline:
- approximately 90% of attempts successfully reached exam start,
- clients significantly below this threshold could be automatically flagged for investigation.

The project also supported interface simplification initiatives and A/B testing of onboarding improvements.

## Technical Highlights
This project involved:
- event log analysis,
- heuristic sessionization,
- reconstruction of incomplete user behavior,
- noisy data filtering,
- validation of reconstructed states,
- product analytics,
- onboarding funnel analysis.
