# Interface for Demand Responsive Transport
Documentation for the Interface for Demand Responsive Transport, a standardized data interface for demand responsive transport systems.

## What is IDRT?
The Interface for Demand Responsive Transport, known as IDRT, is the open data interface standard for demand responsive transport systems. IDRT makes it possible for third-party applications or systems to inform about, to book and to pay for the DRT service. IDRT is an open standard where public, private sector and non-profit organization, application developers, and technology vendors are invited to participate in.

The specification has been designed with the following concepts in mind:
* Provide the availability of DRT-service at this moment
* Make booking of the DRT-service possible
* Do not provide information whose primary purpose is historical

## Overview of the Change Process
IDRT is an open specification, developed and maintained by the community of producers and consumers of IDRT. The specification is not fixed or unchangeable. As the DRT-service evolves, it is expected that the specification will be extended by the community to include new features and capabilities over time. To manage the change process, the following guidelines have been established.

The general outline for changing the spec has 4 steps:
* Propose a change by opening an issue at the IDRT GitHub repository.
* Receive comments and feedback from the community and iterate on the proposed change. Discussion lasts for as long as the proposer feels necessary, but must be at least 30 calendar days
* Submit a final request-for-comments on the proposed change to the issue discussion.
* If no outstanding issues are identified after one week’s time, and there is general agreement that the proposed change is worthwhile and follows the guiding principles outlined below, the proposal will be officially adopted.

## Guiding Principles
To preserve the original vision of IDRT, the following guiding principles should be taken into consideration when proposing extensions to the spec:
* IDRT is a specification for real-time data. The spec is not intended for historical or archival data such as trip records.
* Its primary purpose is to power tools for users that will make DRT-service more accessible.
* Changes to the spec should be backwards-compatible, when possible. Caution should be taken to avoid making changes to the spec that would render previous versions invalid.
* Speculative features are discouraged. Each new addition to the spec adds complexity. We want to avoid additions to the spec that do not provide significant additional value.

## Version History
* v0.1: current Version
  * 2020: first draft

## Copyright
The copyright is held by the [North American Bikeshare Association](https://nabsa.net/).
