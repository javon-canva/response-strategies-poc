# High-level Design Doc: Response Strategies Repo 2.0

**Authors:** JT Javon Taffe  
**Team/Group:** Detection and Response  
**Date:** Jan 22, 2025  
**Status:** In review  
**Due:** 31st Jan 2025  

## Background

The response-strategies repository houses critical playbooks and Standard Operating Procedures (SOPs) that direct the Detection and Response (D&R) incident response process.

Nevertheless, several challenges have arisen:

1. Inconsistent responses and resolutions to alerts and incidents.
2. Redundant content present within playbooks and SOPs.
3. Difficulty in locating pertinent information during incidents.
4. Limited standardisation in the management of common scenarios.
5. Lack of clear standards for when technical content should be separated into standalone artifacts versus embedded directly within documentation, leading to inconsistent formatting and accessibility of critical information.

These challenges adversely affect the efficiency and effectiveness of our incident response process, potentially resulting in prolonged resolution times and inconsistent management of security events. As the Detection and Response (D&R) function evolves and transitions toward greater automation, the establishment of clear processes and procedures will provide the team with a robust foundation for future development.

## Goals

1. Standardise incident response procedures while maintaining flexibility to handle unique scenarios and support future automation
2. Reduce resolution times and inconsistencies by establishing clear, repeatable investigation patterns
3. Eliminate redundancy through modular SOPs that can be referenced across multiple playbooks
4. Improve information find-ability during active incidents through better repository organisation
5. Define clear standards for documentation types, including specific criteria for artefacts versus inline content
6. Create a foundation for automated response capabilities through well-defined, consistent procedures

## Non-goals

- Rewrite all existing playbooks (focus on template and standards for new content)
- Build automated tooling for playbook creation/management
- Replace existing incident management tools or processes
- Create playbooks for every possible scenario

## Recommended Solution

### Description

We propose the implementation of a modular documentation approach designed to enhance our response to alerts, incidents, and other common detection and response (D&R) tasks. This solution encompasses five key document types:

#### 1. Incident Playbooks [Template]
This category includes incident type-specific playbooks (e.g., malware, credential theft) that outline comprehensive response procedures for specific security incidents. Each playbook details investigation steps, containment actions, and remediation guidelines.

#### 2. Data Source-Specific Procedures [Template]
These documents provide complete response protocols tailored for particular cloud platforms or data sources, such as AWS and GCP, focusing on investigating and managing incidents effectively within these environments. In some cases these can also be dashboards related to certain data sources. Given our current split of logs across multiple locations (Elastic, new SentinelOne tenant, old SentinelOne tenant), these procedures will include up-to-date guidance on which platform to query for specific data types, ensuring analysts can quickly access relevant information without wasting time searching across multiple systems.

#### 3. Modular Procedures (Standard Operating Procedures - SOPs) [Template]
This section comprises reusable and standardised procedures that can be referenced across multiple playbooks. Examples include user containment, AWS session termination, and evidence collection.

#### 4. Investigation Patterns [Template]
Here, we outline standard investigative workflows and techniques applicable to various incident types. This includes data correlation methods, common pivot points, and evidence collection requirements.

#### 5. Artefact Creation Guidelines [Template]
This document establishes standards for the creation and maintenance of technical artefacts (such as code, diagrams, and reports) used during incident response. It also delineates when to create artefacts versus inline content.

This modular documentation approach aims to enhance our operational efficiency and ensure effective incident management.

### Pros

- Creates documentation that can be easily automated
- Improves consistency and maintainability
- Reduces duplication through modular approach
- Makes information easier to find during incidents
- Maintains flexibility while enforcing standards

### Cons

- Initial effort required to create new templates and guidelines
- Need to train team on new standards
- May need to update some existing content for consistency
- Additional review overhead to maintain standards

## Risks and Assumptions

### Assumptions
- Assumes team will follow new standards consistently

### Risks
- Risk of over-standardisation limiting flexibility
- May need to adjust standards based on feedback
- Could temporarily slow down content creation during transition

## Dependencies

- Buy-in from D&R team members
- Alignment with existing documentation standards
- Access to repository management tools

## Estimated Effort

**Medium (4-8 weeks)**
- 2 weeks for template and guideline creation
- 2 weeks for review and refinement
- 2-4 weeks for initial implementation and training

## How do we define success

- All new playbooks follow standardised templates
- Reduced content duplication
- Positive feedback from team on usability
- Faster average time to find relevant information
- Maintained or improved incident response times

## Other Solutions Considered

### Option 1: Do Nothing

**Description**  
Continue with current repository structure and processes without implementing standardisation or improvements.

**Pros**
1. No immediate effort required
2. Teams maintain full flexibility in documentation approach
3. No training or process changes needed
4. No risk of disrupting current workflows

**Cons**
1. Content duplication continues to increase
2. Inconsistent quality and structure across playbooks
3. Growing maintenance burden
4. Difficulty finding information during incidents
5. Slower onboarding for new team members
6. Increased risk of missing critical steps during incident response

**Risks and Assumptions**
1. Technical debt will continue to accumulate
2. May need more extensive rework later
3. Could impact incident response effectiveness
4. May lead to inconsistent handling of incidents

**Dependencies**  
None

**Estimated Effort**  
None