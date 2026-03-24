---
id: core-operating-playbook
type: core
owner_role: role.cto
lifecycle_stages:
  - lifecycle.discovery
  - lifecycle.scoping
  - lifecycle.design
  - lifecycle.planning
  - lifecycle.implementation
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
  - lifecycle.incident
  - lifecycle.postmortem
  - lifecycle.deprecation
primary_input_artifacts: []
primary_output_artifacts:
  - artifact.prd
  - artifact.tech-design-doc
  - artifact.architecture-diagram
  - artifact.task
  - artifact.test-plan
  - artifact.test-results
  - artifact.release-checklist
  - artifact.runbook
  - artifact.incident-report
  - artifact.postmortem
  - artifact.deprecation-plan
trigger_events:
  - governance.playbook.create
  - governance.playbook.update
  - governance.audit.run
sla_class: sla.standard
review_cadence: quarterly
status: active
---

# Core Operating Playbook

## 1. System Principles

### rule.invariant.documented-existence
- statement: If it is not documented, it does not exist.
- requirement: Every lifecycle action, artifact, decision, transition, approval, rejection, exception, release, incident, and override MUST be recorded in a referenced artifact or decision log entry before it is treated as valid.
- enforcement: Any work item, transition, or claim without a referenced record is INVALID and MUST be rejected at the next gate.
- validation_method: Verify presence of at least one canonical artifact reference or decision log reference for every asserted work state.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.single-artifact-owner
- statement: Every artifact has an owner.
- requirement: Every artifact instance MUST declare exactly one `owner_role` using a canonical role ID.
- enforcement: Any artifact with zero owners, multiple owners, or a non-canonical owner is INVALID.
- validation_method: Verify one and only one `owner_role` field exists and its value matches a canonical role ID.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.decision-traceability
- statement: Every decision is traceable.
- requirement: Every decision MUST have a unique decision log entry ID and MUST reference the affected artifact IDs and lifecycle stage IDs.
- enforcement: Any decision without a decision log entry, linked artifact reference, or linked lifecycle reference is INVALID.
- validation_method: Verify each decision record includes `decision_id`, `linked_artifact_ids`, `linked_lifecycle_stage_ids`, `owner_role`, `timestamp`, and `outcome`.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.gated-lifecycle-transition
- statement: Every lifecycle transition is gated.
- requirement: Every transition between lifecycle stages MUST include a gate decision with an explicit acceptance state.
- enforcement: Any stage transition without a gate decision is INVALID and MUST NOT advance state.
- validation_method: Verify each transition record includes `from_stage`, `to_stage`, `acceptance_state`, `approver_role`, and referenced handoff artifacts.
- override_policy: NOT OVERRIDABLE.

## 2. Lifecycle State Machine

### lifecycle.discovery
- rule_id: rule.lifecycle.discovery
- purpose: Define the problem, user need, business objective, constraints, and initial success criteria.
- entry_criteria:
  - trigger_event_recorded
- exit_criteria:
  - artifact.prd
- owner_role: role.pm
- allowed_transitions:
  - lifecycle.scoping

### lifecycle.scoping
- rule_id: rule.lifecycle.scoping
- purpose: Convert the documented problem into bounded scope, delivery intent, and initial feasibility.
- entry_criteria:
  - artifact.prd
- exit_criteria:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
- owner_role: role.pm
- allowed_transitions:
  - lifecycle.design

### lifecycle.design
- rule_id: rule.lifecycle.design
- purpose: Produce the technical design, architecture definition, interfaces, and implementation constraints.
- entry_criteria:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
- exit_criteria:
  - artifact.task
  - artifact.test-plan
- owner_role: role.backend
- allowed_transitions:
  - lifecycle.planning

### lifecycle.planning
- rule_id: rule.lifecycle.planning
- purpose: Convert the approved design into executable tasks, test intent, sequencing, and ownership.
- entry_criteria:
  - artifact.task
  - artifact.test-plan
- exit_criteria:
  - artifact.task
  - artifact.test-plan
- owner_role: role.em
- allowed_transitions:
  - lifecycle.implementation

### lifecycle.implementation
- rule_id: rule.lifecycle.implementation
- purpose: Produce the implementation represented by planned tasks and preserve traceability to design and test intent.
- entry_criteria:
  - artifact.task
  - artifact.test-plan
- exit_criteria:
  - artifact.test-results
  - artifact.runbook
- owner_role: role.backend
- allowed_transitions:
  - lifecycle.testing

### lifecycle.testing
- rule_id: rule.lifecycle.testing
- purpose: Validate implemented behavior against the test plan, quality gates, and release readiness requirements.
- entry_criteria:
  - artifact.test-results
  - artifact.runbook
- exit_criteria:
  - artifact.test-results
  - artifact.runbook
- owner_role: role.qa
- allowed_transitions:
  - lifecycle.release

### lifecycle.release
- rule_id: rule.lifecycle.release
- purpose: Approve production delivery, rollback readiness, and operational ownership transfer.
- entry_criteria:
  - artifact.test-results
  - artifact.runbook
- exit_criteria:
  - artifact.release-checklist
  - artifact.runbook
- owner_role: role.devops
- allowed_transitions:
  - lifecycle.operations
  - lifecycle.incident

### lifecycle.operations
- rule_id: rule.lifecycle.operations
- purpose: Operate the released system, maintain service health, support users, and manage change feedback.
- entry_criteria:
  - artifact.release-checklist
  - artifact.runbook
- exit_criteria:
  - artifact.release-checklist
  - artifact.runbook
- owner_role: role.devops
- allowed_transitions:
  - lifecycle.incident
  - lifecycle.deprecation

### lifecycle.incident
- rule_id: rule.lifecycle.incident
- purpose: Record, triage, mitigate, and communicate production failures or high-severity operational risks.
- entry_criteria:
  - artifact.runbook
- exit_criteria:
  - artifact.incident-report
- owner_role: role.devops
- allowed_transitions:
  - lifecycle.postmortem

### lifecycle.postmortem
- rule_id: rule.lifecycle.postmortem
- purpose: Determine root cause, corrective actions, and system changes required after an incident.
- entry_criteria:
  - artifact.incident-report
- exit_criteria:
  - artifact.postmortem
- owner_role: role.em
- allowed_transitions:
  - lifecycle.operations

### lifecycle.deprecation
- rule_id: rule.lifecycle.deprecation
- purpose: Remove or retire a system, capability, or workflow under a documented retirement plan.
- entry_criteria:
  - artifact.release-checklist
  - artifact.runbook
- exit_criteria:
  - artifact.deprecation-plan
- owner_role: role.pm
- allowed_transitions: []

### rule.lifecycle.transition-equality
- statement: Exit criteria of stage N MUST equal entry criteria of stage N+1 for the main lifecycle chain.
- main_chain:
  - lifecycle.discovery -> lifecycle.scoping
  - lifecycle.scoping -> lifecycle.design
  - lifecycle.design -> lifecycle.planning
  - lifecycle.planning -> lifecycle.implementation
  - lifecycle.implementation -> lifecycle.testing
  - lifecycle.testing -> lifecycle.release
  - lifecycle.release -> lifecycle.operations
- boundary_sets:
  - lifecycle.discovery.exit == lifecycle.scoping.entry == [artifact.prd]
  - lifecycle.scoping.exit == lifecycle.design.entry == [artifact.tech-design-doc, artifact.architecture-diagram]
  - lifecycle.design.exit == lifecycle.planning.entry == [artifact.task, artifact.test-plan]
  - lifecycle.planning.exit == lifecycle.implementation.entry == [artifact.task, artifact.test-plan]
  - lifecycle.implementation.exit == lifecycle.testing.entry == [artifact.test-results, artifact.runbook]
  - lifecycle.testing.exit == lifecycle.release.entry == [artifact.test-results, artifact.runbook]
  - lifecycle.release.exit == lifecycle.operations.entry == [artifact.release-checklist, artifact.runbook]
- enforcement: Any main-chain transition with a non-matching boundary set is INVALID.

## 3. Artifact Contracts

### artifact.prd
- rule_id: rule.artifact.prd
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - problem_statement
  - target_outcomes
  - scope_constraints
  - acceptance_criteria
- lifecycle_stage_created: lifecycle.discovery
- lifecycle_stage_consumed:
  - lifecycle.scoping
- validation_rules:
  - artifact_id MUST equal artifact.prd.
  - owner_role MUST equal role.pm.
  - source_lifecycle_stage MUST equal lifecycle.discovery.
  - target_lifecycle_stage MUST equal lifecycle.scoping.
  - status MUST be one of [draft, active, deprecated, under-review].
  - linked_decision_ids MUST contain at least one decision ID.
  - problem_statement, target_outcomes, scope_constraints, and acceptance_criteria MUST be non-empty.

### artifact.tech-design-doc
- rule_id: rule.artifact.tech-design-doc
- owner_role: role.backend
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - requirements_reference
  - system_constraints
  - interface_definitions
  - dependency_map
  - risk_register
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.planning
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.tech-design-doc.
  - owner_role MUST equal role.backend.
  - source_lifecycle_stage MUST equal lifecycle.design.
  - target_lifecycle_stage MUST include lifecycle.planning.
  - target_lifecycle_stage MUST include lifecycle.implementation.
  - requirements_reference MUST reference artifact.prd.
  - interface_definitions and dependency_map MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.architecture-diagram
- rule_id: rule.artifact.architecture-diagram
- owner_role: role.backend
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - system_boundaries
  - component_nodes
  - data_flows
  - trust_boundaries
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.planning
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.architecture-diagram.
  - owner_role MUST equal role.backend.
  - source_lifecycle_stage MUST equal lifecycle.design.
  - target_lifecycle_stage MUST include lifecycle.planning.
  - target_lifecycle_stage MUST include lifecycle.implementation.
  - component_nodes and data_flows MUST be non-empty.
  - trust_boundaries MUST be explicit.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.task
- rule_id: rule.artifact.task
- owner_role: role.em
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - parent_design_references
  - task_scope
  - assignee_role
  - definition_of_done
  - dependency_references
- lifecycle_stage_created: lifecycle.planning
- lifecycle_stage_consumed:
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.task.
  - owner_role MUST equal role.em.
  - source_lifecycle_stage MUST equal lifecycle.planning.
  - target_lifecycle_stage MUST equal lifecycle.implementation.
  - assignee_role MUST be a canonical role ID.
  - parent_design_references MUST reference artifact.tech-design-doc or artifact.architecture-diagram.
  - definition_of_done MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.test-plan
- rule_id: rule.artifact.test-plan
- owner_role: role.qa
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - scope_reference
  - test_scenarios
  - environment_requirements
  - pass_fail_criteria
  - observability_requirements
- lifecycle_stage_created: lifecycle.planning
- lifecycle_stage_consumed:
  - lifecycle.testing
- validation_rules:
  - artifact_id MUST equal artifact.test-plan.
  - owner_role MUST equal role.qa.
  - source_lifecycle_stage MUST equal lifecycle.planning.
  - target_lifecycle_stage MUST equal lifecycle.testing.
  - test_scenarios MUST be non-empty.
  - pass_fail_criteria MUST be non-empty.
  - observability_requirements MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.test-results
- rule_id: rule.artifact.test-results
- owner_role: role.qa
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - test_plan_reference
  - execution_summary
  - failed_cases
  - evidence_links
  - release_recommendation
- lifecycle_stage_created: lifecycle.testing
- lifecycle_stage_consumed:
  - lifecycle.release
- validation_rules:
  - artifact_id MUST equal artifact.test-results.
  - owner_role MUST equal role.qa.
  - source_lifecycle_stage MUST equal lifecycle.testing.
  - target_lifecycle_stage MUST equal lifecycle.release.
  - test_plan_reference MUST reference artifact.test-plan.
  - execution_summary and evidence_links MUST be non-empty.
  - release_recommendation MUST be one of [accepted, rejected, needs-revision].
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.release-checklist
- rule_id: rule.artifact.release-checklist
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - release_scope
  - deployment_steps
  - rollback_steps
  - approval_evidence
  - operational_readiness
- lifecycle_stage_created: lifecycle.release
- lifecycle_stage_consumed:
  - lifecycle.operations
  - lifecycle.deprecation
- validation_rules:
  - artifact_id MUST equal artifact.release-checklist.
  - owner_role MUST equal role.devops.
  - source_lifecycle_stage MUST equal lifecycle.release.
  - target_lifecycle_stage MUST include lifecycle.operations.
  - rollback_steps MUST be non-empty.
  - approval_evidence MUST include acceptance_state and approver_role.
  - operational_readiness MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.runbook
- rule_id: rule.artifact.runbook
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - service_overview
  - observability_endpoints
  - escalation_steps
  - rollback_reference
  - recovery_steps
- lifecycle_stage_created: lifecycle.implementation
- lifecycle_stage_consumed:
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
  - lifecycle.incident
- validation_rules:
  - artifact_id MUST equal artifact.runbook.
  - owner_role MUST equal role.devops.
  - source_lifecycle_stage MUST equal lifecycle.implementation.
  - target_lifecycle_stage MUST include lifecycle.testing.
  - target_lifecycle_stage MUST include lifecycle.release.
  - target_lifecycle_stage MUST include lifecycle.operations.
  - target_lifecycle_stage MUST include lifecycle.incident.
  - observability_endpoints, rollback_reference, and recovery_steps MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.incident-report
- rule_id: rule.artifact.incident-report
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - severity
  - impact_window
  - affected_services
  - mitigation_actions
  - timeline
- lifecycle_stage_created: lifecycle.incident
- lifecycle_stage_consumed:
  - lifecycle.postmortem
- validation_rules:
  - artifact_id MUST equal artifact.incident-report.
  - owner_role MUST equal role.devops.
  - source_lifecycle_stage MUST equal lifecycle.incident.
  - target_lifecycle_stage MUST equal lifecycle.postmortem.
  - severity MUST be non-empty.
  - timeline MUST be non-empty.
  - mitigation_actions MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.postmortem
- rule_id: rule.artifact.postmortem
- owner_role: role.em
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - incident_reference
  - root_cause
  - corrective_actions
  - preventive_actions
  - closure_owner_role
- lifecycle_stage_created: lifecycle.postmortem
- lifecycle_stage_consumed:
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.postmortem.
  - owner_role MUST equal role.em.
  - source_lifecycle_stage MUST equal lifecycle.postmortem.
  - target_lifecycle_stage MUST equal lifecycle.operations.
  - incident_reference MUST reference artifact.incident-report.
  - root_cause, corrective_actions, and preventive_actions MUST be non-empty.
  - closure_owner_role MUST be a canonical role ID.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.deprecation-plan
- rule_id: rule.artifact.deprecation-plan
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - retirement_scope
  - retirement_date
  - migration_path
  - dependency_impact
  - approval_evidence
- lifecycle_stage_created: lifecycle.deprecation
- lifecycle_stage_consumed:
  - lifecycle.deprecation
- validation_rules:
  - artifact_id MUST equal artifact.deprecation-plan.
  - owner_role MUST equal role.pm.
  - source_lifecycle_stage MUST equal lifecycle.deprecation.
  - target_lifecycle_stage MUST equal lifecycle.deprecation.
  - retirement_scope, retirement_date, and migration_path MUST be non-empty.
  - dependency_impact MUST be non-empty.
  - approval_evidence MUST include acceptance_state and approver_role.
  - linked_decision_ids MUST contain at least one decision ID.

## 4. SLA System

### sla.standard
- rule_id: rule.sla.standard
- response_time: 1-business-day
- completion_time: 5-business-days
- escalation_trigger: response_time_exceeded

### sla.expedited
- rule_id: rule.sla.expedited
- response_time: 4-hours
- completion_time: 2-business-days
- escalation_trigger: response_time_exceeded_or_deadline_risk_confirmed

### sla.emergency
- rule_id: rule.sla.emergency
- response_time: 15-minutes
- completion_time: 24-hours
- escalation_trigger: immediate_on_assignment

### rule.sla.custom-definition-prohibited
- statement: Roles, workflows, artifacts, and inherited playbooks MUST reference an existing SLA class and MUST NOT define a custom SLA.
- enforcement: Any playbook that defines an SLA outside [sla.standard, sla.expedited, sla.emergency] is INVALID.

## 5. Decision Framework

### rule.decision.daci-model
- decision_model: DACI
- roles:
  - driver: role.em
  - approver: role.cto
  - contributor:
      - role.pm
      - role.backend
      - role.frontend
      - role.qa
      - role.devops
      - role.security
      - role.design
      - role.data
  - informed:
      - role.em
      - role.pm
      - role.devops
- canonical_approver_model: A decision MUST have exactly one approver role per decision record.

### rule.decision.threshold-minor
- threshold_id: THRESHOLD_MINOR
- requirement: Changes classified at or below THRESHOLD_MINOR MUST be approved by the designated approver for the impacted workflow or artifact.
- enforcement: A decision record without threshold classification is INVALID.

### rule.decision.threshold-major
- threshold_id: THRESHOLD_MAJOR
- requirement: Changes classified at or above THRESHOLD_MAJOR MUST be approved by role.cto.
- enforcement: A major decision without role.cto approval is INVALID.

### rule.decision.conflict-escalation
- statement: Equal-level disagreement MUST escalate to the next level role archetype.
- protocol:
  - role.backend and role.frontend disagreement -> role.em
  - role.pm and role.em disagreement -> role.cto
  - role.qa and role.devops disagreement -> role.em
  - role.security and role.data disagreement -> role.cto
- enforcement: A blocked decision without escalation record is INVALID.

## 6. Handoff Contract System

### Discovery -> Scoping
- rule_id: rule.handoff.discovery-to-scoping
- required_artifacts:
  - artifact.prd
- validation_conditions:
  - artifact.prd MUST be active or under-review.
  - artifact.prd MUST include owner_role and linked_decision_ids.
  - gate record MUST include from_stage lifecycle.discovery and to_stage lifecycle.scoping.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.pm.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - non-canonical owner_role
  - empty acceptance_criteria in artifact.prd

### Scoping -> Design
- rule_id: rule.handoff.scoping-to-design
- required_artifacts:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
- validation_conditions:
  - both artifacts MUST reference artifact.prd.
  - both artifacts MUST include linked_decision_ids.
  - gate record MUST include from_stage lifecycle.scoping and to_stage lifecycle.design.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.backend.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty interface_definitions
  - empty component_nodes or data_flows

### Design -> Implementation
- rule_id: rule.handoff.design-to-implementation
- required_artifacts:
  - artifact.task
  - artifact.test-plan
- validation_conditions:
  - artifact.task MUST reference artifact.tech-design-doc or artifact.architecture-diagram.
  - artifact.test-plan MUST include observability_requirements.
  - gate record MUST include from_stage lifecycle.design and to_stage lifecycle.planning or lifecycle.implementation.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.em.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty definition_of_done
  - empty observability_requirements

### Implementation -> Testing
- rule_id: rule.handoff.implementation-to-testing
- required_artifacts:
  - artifact.test-results
  - artifact.runbook
- validation_conditions:
  - artifact.test-results MUST reference artifact.test-plan.
  - artifact.runbook MUST include observability_endpoints and rollback_reference.
  - gate record MUST include from_stage lifecycle.implementation and to_stage lifecycle.testing.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.qa.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty evidence_links
  - empty observability_endpoints

### Testing -> Release
- rule_id: rule.handoff.testing-to-release
- required_artifacts:
  - artifact.test-results
  - artifact.runbook
- validation_conditions:
  - artifact.test-results.release_recommendation MUST equal accepted.
  - artifact.runbook MUST include escalation_steps and recovery_steps.
  - gate record MUST include from_stage lifecycle.testing and to_stage lifecycle.release.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.devops.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - release_recommendation not accepted
  - empty recovery_steps

### Release -> Operations
- rule_id: rule.handoff.release-to-operations
- required_artifacts:
  - artifact.release-checklist
  - artifact.runbook
- validation_conditions:
  - artifact.release-checklist MUST include rollback_steps and operational_readiness.
  - artifact.runbook MUST be active.
  - gate record MUST include from_stage lifecycle.release and to_stage lifecycle.operations.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.devops.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty rollback_steps
  - empty operational_readiness

### Incident -> Postmortem
- rule_id: rule.handoff.incident-to-postmortem
- required_artifacts:
  - artifact.incident-report
- validation_conditions:
  - artifact.incident-report MUST include severity, impact_window, mitigation_actions, and timeline.
  - gate record MUST include from_stage lifecycle.incident and to_stage lifecycle.postmortem.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.em.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty timeline
  - empty mitigation_actions

### Support/Feedback -> Roadmap
- rule_id: rule.handoff.support-feedback-to-roadmap
- required_artifacts:
  - artifact.prd
- validation_conditions:
  - artifact.prd MUST be updated with a linked decision ID representing feedback intake.
  - gate record MUST include acceptance_state and owner_role.
  - gate record MUST identify target planning cycle.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST equal role.pm.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - missing linked decision ID
  - missing target planning cycle

### rule.handoff.validity
- statement: No handoff is valid without artifact reference and explicit acceptance state.
- required_fields:
  - artifact_reference
  - acceptance_state
- enforcement: Any handoff missing either field is INVALID.

## 7. Minimum Viable Handoff

### rule.mvh.definition
- artifact_exists: REQUIRED
- owner_assigned: REQUIRED
- accepted_by_next_role: REQUIRED
- invalid_conditions:
  - missing_artifact
  - missing_owner_role
  - missing_acceptance_by_next_role
- enforcement: Any transfer missing one or more REQUIRED MVH conditions is INVALID.

## 8. Process Activation Model

### activation.1-15
- rule_id: rule.activation.1-15
- team_size: 1-15
- enforcement_depth: minimal required artifact set only
- required_reviews: owner_role review plus one approver for THRESHOLD_MAJOR only
- audit_requirements: gate records and decision log entries REQUIRED; full artifact history OPTIONAL unless triggered by incident or release

### activation.16-75
- rule_id: rule.activation.16-75
- team_size: 16-75
- enforcement_depth: full lifecycle gate enforcement
- required_reviews: owner_role review plus approver review for all lifecycle exits and THRESHOLD_MINOR or greater decisions
- audit_requirements: artifact history, gate records, and decision log references REQUIRED for all lifecycle transitions

### activation.76-200
- rule_id: rule.activation.76-200
- team_size: 76-200
- enforcement_depth: strict full-system enforcement
- required_reviews: owner_role review, approver review, and independent reviewer evidence for release, incident, postmortem, and override actions
- audit_requirements: complete audit trail REQUIRED for all artifacts, decisions, transitions, and override records

## 9. Quality Bar

### rule.quality.observability-before-implementation
- statement: No implementation without observability.
- requirement: artifact.test-plan MUST include observability_requirements and artifact.runbook MUST include observability_endpoints before implementation is accepted.
- enforcement: Any implementation accepted without both observability records is INVALID.

### rule.quality.rollback-before-release
- statement: No release without rollback.
- requirement: artifact.release-checklist MUST include rollback_steps and artifact.runbook MUST include rollback_reference before release is accepted.
- enforcement: Any release without documented rollback evidence is INVALID.

### rule.quality.postmortem-after-incident
- statement: No incident without postmortem.
- requirement: Every artifact.incident-report MUST transition to artifact.postmortem.
- enforcement: Any closed incident without artifact.postmortem is INVALID.

### rule.quality.owner-required
- statement: No artifact without owner.
- requirement: Every artifact instance MUST declare exactly one owner_role.
- enforcement: Any ownerless artifact is INVALID.

### rule.quality.decision-log-required
- statement: No decision without log entry.
- requirement: Every decision MUST have a unique decision record and linked artifact references.
- enforcement: Any undocumented decision is INVALID.

### rule.quality.system-validity
- statement: This playbook system is valid only when all mandatory structures are present and consistent.
- measurable_conditions:
  - frontmatter schema completeness == 100 percent
  - lifecycle stage coverage == 11 of 11 canonical stages
  - artifact contract coverage == 11 of 11 canonical artifacts
  - handoff contract coverage == 8 of 8 mandatory handoffs
  - duplicate canonical IDs == 0
  - conflicting lifecycle boundary sets == 0

### rule.quality.system-health
- statement: This playbook system is healthy only when operational compliance remains above the minimum thresholds.
- measurable_conditions:
  - artifact completeness rate >= 99 percent
  - gate compliance rate == 100 percent
  - decision traceability coverage == 100 percent
  - handoff acceptance rate >= 95 percent
  - incident postmortem closure rate == 100 percent

## 10. Anti-Patterns

### anti-pattern.01-undocumented-work
- description: Work is advanced without a referenced artifact or decision log entry.
- detection_signal: lifecycle transition record exists with no artifact_reference and no decision_id.
- consequence: transition is INVALID and MUST be rolled back to the prior valid stage.

### anti-pattern.02-ownerless-artifact
- description: An artifact exists with no owner_role or more than one owner_role.
- detection_signal: artifact metadata contains zero or multiple owner_role fields.
- consequence: artifact is INVALID and MUST be rejected from all handoffs.

### anti-pattern.03-unlogged-decision
- description: A decision changes scope, design, release, or operations state without a decision record.
- detection_signal: state change exists with no linked_decision_ids.
- consequence: decision is INVALID and dependent artifacts MUST be marked needs-revision.

### anti-pattern.04-ungated-transition
- description: A lifecycle stage changes without explicit acceptance_state.
- detection_signal: from_stage and to_stage are recorded without acceptance_state.
- consequence: transition is INVALID and MUST NOT be recognized by downstream stages.

### anti-pattern.05-noncanonical-id
- description: A lifecycle, artifact, role, or SLA value uses a synonym or custom string.
- detection_signal: ID value is not present in the canonical ID set.
- consequence: record is INVALID and MUST be corrected before processing.

### anti-pattern.06-design-without-traceability
- description: A design artifact does not reference its originating requirements artifact.
- detection_signal: artifact.tech-design-doc or artifact.architecture-diagram has no reference to artifact.prd.
- consequence: design handoff is INVALID and MUST be rejected.

### anti-pattern.07-testing-without-observability
- description: Testing proceeds without observability requirements or endpoints.
- detection_signal: artifact.test-plan.observability_requirements is empty or artifact.runbook.observability_endpoints is empty.
- consequence: testing handoff is INVALID and release eligibility is denied.

### anti-pattern.08-release-without-rollback
- description: Release approval is attempted without rollback documentation.
- detection_signal: artifact.release-checklist.rollback_steps is empty or artifact.runbook.rollback_reference is empty.
- consequence: release is INVALID and MUST be blocked.

### anti-pattern.09-closed-incident-without-postmortem
- description: An incident is marked resolved without a postmortem artifact.
- detection_signal: artifact.incident-report status indicates closure and no artifact.postmortem reference exists.
- consequence: incident closure is INVALID and operational compliance is failed.

### anti-pattern.10-handoff-without-acceptance
- description: A handoff package is delivered without explicit receiver acceptance state.
- detection_signal: handoff record missing acceptance_state or accepting_role.
- consequence: handoff is INVALID and ownership does not transfer.

### anti-pattern.11-override-without-replacement
- description: A downstream playbook claims an override but does not define a replacement rule.
- detection_signal: override record cites a rule ID but lacks replacement_rule.
- consequence: override is INVALID and parent rule remains in force.

### anti-pattern.12-conflicting-boundary-set
- description: Adjacent lifecycle stages define non-matching entry and exit artifact sets on the main chain.
- detection_signal: lifecycle boundary comparison returns inequality.
- consequence: lifecycle specification is INVALID and must be corrected before inheritance.

## 11. Inheritance & Override Rules

### rule.inheritance.required
- statement: All playbooks MUST inherit this document.
- requirement: Every playbook frontmatter MUST reference `core-operating-playbook` as its governing parent.
- enforcement: Any playbook without inheritance reference is INVALID.

### rule.override.reference-exact-rule
- statement: Every override MUST reference the exact parent rule ID being overridden.
- requirement: override records MUST include `target_rule_id`.
- enforcement: An override without `target_rule_id` is INVALID.

### rule.override.require-justification
- statement: Every override MUST justify deviation.
- requirement: override records MUST include `justification`.
- enforcement: An override without justification is INVALID.

### rule.override.require-replacement
- statement: Every override MUST define the replacement rule.
- requirement: override records MUST include `replacement_rule`.
- enforcement: An override without replacement_rule is INVALID.

### rule.override.invalid-without-all-elements
- statement: Overrides without exact rule reference, justification, and replacement rule are INVALID.
- required_fields:
  - target_rule_id
  - justification
  - replacement_rule
- enforcement: Missing any required field invalidates the override and the parent rule remains authoritative.

## 12. Machine-Readable Guarantees

### rule.machine.fixed-section-structure
- statement: Section order is fixed and parsable.
- required_order:
  - 1. System Principles
  - 2. Lifecycle State Machine
  - 3. Artifact Contracts
  - 4. SLA System
  - 5. Decision Framework
  - 6. Handoff Contract System
  - 7. Minimum Viable Handoff
  - 8. Process Activation Model
  - 9. Quality Bar
  - 10. Anti-Patterns
  - 11. Inheritance & Override Rules
  - 12. Machine-Readable Guarantees
  - 13. Governance
  - 14. Validation Criteria
- enforcement: Any inherited playbook that reorders required sections is INVALID.

### rule.machine.canonical-ids-only
- statement: All IDs MUST use canonical strings only.
- id_sets:
  - lifecycle: [lifecycle.discovery, lifecycle.scoping, lifecycle.design, lifecycle.planning, lifecycle.implementation, lifecycle.testing, lifecycle.release, lifecycle.operations, lifecycle.incident, lifecycle.postmortem, lifecycle.deprecation]
  - artifact: [artifact.prd, artifact.tech-design-doc, artifact.architecture-diagram, artifact.task, artifact.test-plan, artifact.test-results, artifact.release-checklist, artifact.runbook, artifact.incident-report, artifact.postmortem, artifact.deprecation-plan]
  - role: [role.cto, role.pm, role.em, role.backend, role.frontend, role.qa, role.devops, role.security, role.design, role.data]
  - sla: [sla.standard, sla.expedited, sla.emergency]
- enforcement: Any non-canonical ID is INVALID.

### rule.machine.traceable-relationships
- statement: Lifecycle-to-artifact and role-to-artifact relationships MUST be explicit and traceable.
- required_relationship_fields:
  - source_lifecycle_stage
  - target_lifecycle_stage
  - owner_role
  - linked_decision_ids
- enforcement: Any artifact contract or artifact instance without explicit relationships is INVALID.

### rule.machine.explicit-cross-references
- statement: Cross-references MUST be explicit and MUST NOT be implied.
- requirement: Every dependency, predecessor, successor, parent artifact, and decision reference MUST be declared by canonical ID.
- enforcement: Implicit references are INVALID.

### rule.machine.stable-rule-ids
- statement: Every subsection MUST expose a stable rule ID or canonical ID label suitable for override targeting.
- requirement: Each repeatable block MUST begin with `rule_id` or use a canonical ID heading.
- enforcement: A block without stable identifier is INVALID.

## 13. Governance

### rule.governance.owner
- owner_role: role.cto
- authority_scope: core-operating-playbook
- enforcement: Governance changes without role.cto ownership are INVALID.

### rule.governance.update-process
- required_steps:
  - propose change with decision log entry
  - update affected rule IDs and cross-references
  - validate no duplicate or conflicting canonical IDs
  - obtain approval from role.cto
  - publish new semantic version
- enforcement: Any update missing one or more required steps is INVALID.

### rule.governance.versioning
- versioning_scheme: semantic-versioning
- major_change_definition: any change that alters canonical IDs, required schema fields, lifecycle structure, artifact contracts, or override semantics
- minor_change_definition: any additive rule, validation, or clarification that does not break inherited playbooks
- patch_change_definition: any corrective text update that does not change meaning or enforcement
- enforcement: Version increments MUST match the highest-impact rule change in the update set.

### rule.governance.backward-compatibility
- statement: Minor and patch versions MUST preserve compatibility for inheriting playbooks; major versions MAY require explicit migration.
- requirement: Any breaking governance change MUST include migration guidance in the approving decision log entry.
- enforcement: A breaking change without major version increment is INVALID.

### rule.governance.superseded-rule-deprecation
- statement: A superseded rule MUST remain referenceable until all inheriting playbooks migrate.
- requirement: superseded rules MUST be marked deprecated, reference successor rule IDs, and retain historical traceability.
- enforcement: Removing a referenced rule without successor mapping is INVALID.

## 14. Validation Criteria

### rule.validation.lifecycle-connected
- requirement: All lifecycle stages MUST be connected through allowed transitions.
- validation_method: Verify every canonical lifecycle ID appears in Section 2 and participates in at least one incoming or outgoing transition except lifecycle.discovery incoming and lifecycle.deprecation outgoing.

### rule.validation.artifacts-referenced
- requirement: All artifact IDs MUST be referenced by at least one lifecycle stage, one artifact contract, and one handoff or quality rule.
- validation_method: Verify full artifact coverage across Sections 2, 3, 6, and 9.

### rule.validation.handoffs-defined
- requirement: All eight mandatory handoffs MUST be defined exactly once.
- validation_method: Verify one and only one block exists for each required handoff contract.

### rule.validation.no-conflicting-ids
- requirement: No duplicate or conflicting canonical IDs MAY exist.
- validation_method: Verify uniqueness across lifecycle IDs, artifact IDs, role IDs, SLA IDs, rule IDs, and anti-pattern IDs.

### rule.validation.yaml-schema-consistent
- requirement: YAML frontmatter MUST contain all required schema keys exactly once and MUST use canonical values.
- validation_method: Verify keys [id, type, owner_role, lifecycle_stages, primary_input_artifacts, primary_output_artifacts, trigger_events, sla_class, review_cadence, status] are present exactly once.
