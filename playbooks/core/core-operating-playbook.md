---
id: core-operating-playbook
type: core
owner_role: role.cto
governing_parent: none
related_playbooks: []
related_lifecycle_namespaces:
  - people-lifecycle.hiring
  - people-lifecycle.onboarding
lifecycle_stages:
  - lifecycle.bootstrap
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
  - artifact.okr
  - artifact.roadmap
  - artifact.sprint-plan
  - artifact.support-escalation
  - artifact.analytics-spec
  - artifact.dashboard
  - artifact.hiring-scorecard
  - artifact.onboarding-checklist
  - artifact.vendor-eval
  - artifact.security-review
  - artifact.privacy-impact
  - artifact.budget-request
  - artifact.decision-log
  - artifact.event-log
  - artifact.role-registry
  - artifact.artifact-registry
  - artifact.sla-registry
trigger_events:
  - governance.playbook.create
  - governance.playbook.update
  - governance.audit.run
  - support.feedback.received
  - operations.incident.detected
  - system.bootstrap.requested
sla_class: sla.standard
review_cadence: quarterly
status: active
---

# Core Operating Playbook

## 1. System Principles

### rule.invariant.documented-existence
- statement: If it is not documented, it does not exist.
- requirement: Every lifecycle action, artifact, decision, transition, approval, rejection, exception, release, incident, override, event, and registration MUST be recorded in a referenced artifact or decision record before it is treated as valid.
- enforcement: Any work item, transition, or claim without a referenced record is INVALID and MUST be blocked by the enforcement engine.
- validation_method: Verify presence of at least one canonical artifact reference or `artifact.decision-log` reference for every asserted work state.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.single-artifact-owner
- statement: Every artifact has an owner.
- requirement: Every artifact instance MUST declare exactly one `owner_role` using a locked core role ID or a registered extension role ID.
- enforcement: Any artifact with zero owners, multiple owners, or an unregistered owner is INVALID.
- validation_method: Verify one and only one `owner_role` field exists and its value is registered under `rule.machine.canonical-ids-only`.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.decision-traceability
- statement: Every decision is traceable.
- requirement: Every decision MUST be recorded as `artifact.decision-log` and MUST reference affected artifact IDs, lifecycle stage IDs, version IDs, and event IDs where applicable.
- enforcement: Any decision without a decision artifact, linked artifact reference, linked lifecycle reference, linked version reference, or linked event reference is INVALID.
- validation_method: Verify each decision record includes `decision_id`, `linked_artifact_ids`, `linked_lifecycle_stage_ids`, `linked_versions`, `linked_event_ids`, `owner_role`, `timestamp`, and `outcome`.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.gated-lifecycle-transition
- statement: Every lifecycle transition is gated.
- requirement: Every transition between lifecycle stages MUST include a gate decision with an explicit acceptance state.
- enforcement: Any stage transition without a gate decision is INVALID and MUST NOT advance state.
- validation_method: Verify each transition record includes `from_stage`, `to_stage`, `acceptance_state`, `approver_role`, and referenced handoff artifacts.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.customer-impact-accounted
- statement: Customer impact MUST be explicit.
- requirement: Any artifact or decision affecting customer experience, support obligations, privacy, security, or external commitments MUST declare customer impact in the governing artifact or decision record.
- enforcement: A change with undeclared customer impact is INVALID.
- validation_method: Verify affected artifacts contain `customer_impact` or equivalent declared field where applicable.
- override_policy: NOT OVERRIDABLE.

### rule.invariant.security-default
- statement: Security and privacy review are mandatory for security-relevant change classes.
- requirement: Any change classified as security-sensitive, privacy-impacting, vendor-integrating, or externally exposing data MUST reference `artifact.security-review` or `artifact.privacy-impact` before release acceptance.
- enforcement: Any release lacking required review artifacts for relevant change classes is INVALID.
- validation_method: Verify security-sensitive releases include the required review artifacts and linked decision records.
- override_policy: NOT OVERRIDABLE.

## 2. Lifecycle State Machine

### lifecycle.bootstrap
- rule_id: rule.lifecycle.bootstrap
- purpose: Initialize the operating system, seed foundational registries, establish the event store, and create the first decision records required for all downstream validation.
- runs_once: true
- entry_criteria:
  - governing_parent == none
  - repository_initialization_event_recorded
- exit_criteria:
  - artifact.role-registry
  - artifact.artifact-registry
  - artifact.sla-registry
  - artifact.event-log
  - artifact.decision-log
- owner_role: role.cto
- owner_binding: Governing system owner for first-run initialization.
- allowed_transitions:
  - lifecycle.discovery

### lifecycle.discovery
- rule_id: rule.lifecycle.discovery
- purpose: Define the problem, intake source, business objective, customer impact, and initial success criteria.
- entry_criteria:
  - artifact.role-registry
  - artifact.artifact-registry
  - artifact.sla-registry
  - artifact.event-log
  - artifact.decision-log
- exit_criteria:
  - artifact.prd
- owner_role: role.pm
- owner_binding: Registered domain owner for the opportunity or change request.
- allowed_transitions:
  - lifecycle.scoping

### lifecycle.scoping
- rule_id: rule.lifecycle.scoping
- purpose: Bound the work, classify risk, classify threshold, determine execution mode, and decide whether the change follows standard, small-change, or incident-driven execution.
- entry_criteria:
  - artifact.prd.status == active OR artifact.prd.status == under-review
- exit_criteria:
  - artifact.prd.status == active
  - artifact.prd.scope_constraints != empty
  - artifact.prd.threshold_classification != empty
  - artifact.prd.customer_impact != empty
  - artifact.prd.execution_mode != empty
- owner_role: role.pm
- owner_binding: Registered product or business owner for the scoped change.
- allowed_transitions:
  - lifecycle.design

### lifecycle.design
- rule_id: rule.lifecycle.design
- purpose: Produce the technical design, architecture definition, interfaces, risk treatment, and non-functional constraints.
- entry_criteria:
  - artifact.prd
- exit_criteria:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
- owner_role: role.backend
- owner_binding: Registered engineering domain owner responsible for the design outcome.
- allowed_transitions:
  - lifecycle.planning

### lifecycle.planning
- rule_id: rule.lifecycle.planning
- purpose: Convert approved design into executable sequencing, ownership, dependencies, and test readiness.
- entry_criteria:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
- exit_criteria:
  - artifact.task
  - artifact.test-plan
  - artifact.sprint-plan
- owner_role: role.em
- owner_binding: Registered delivery owner accountable for sequencing and readiness.
- allowed_transitions:
  - lifecycle.implementation

### lifecycle.implementation
- rule_id: rule.lifecycle.implementation
- purpose: Produce the implementation, operational instructions, and execution evidence required for formal validation.
- entry_criteria:
  - artifact.task
  - artifact.test-plan
  - artifact.sprint-plan
- exit_criteria:
  - artifact.test-results
  - artifact.runbook
- owner_role: role.backend
- owner_binding: Registered implementation owner for the affected domain.
- allowed_transitions:
  - lifecycle.testing

### lifecycle.testing
- rule_id: rule.lifecycle.testing
- purpose: Validate behavior, quality, and security readiness using the approved test plan and review artifacts.
- entry_criteria:
  - artifact.test-results.status == under-review
  - artifact.runbook.status == under-review OR artifact.runbook.status == active
- exit_criteria:
  - artifact.test-results.status == active
  - artifact.security-review.status == active
  - artifact.test-results.release_recommendation == accepted
- owner_role: role.qa
- owner_binding: Registered quality owner; implementing domains remain REQUIRED contributors.
- allowed_transitions:
  - lifecycle.release

### lifecycle.release
- rule_id: rule.lifecycle.release
- purpose: Approve production delivery, rollback readiness, security clearance, and operational transfer.
- entry_criteria:
  - artifact.test-results
  - artifact.runbook
  - artifact.security-review
- exit_criteria:
  - artifact.release-checklist
  - artifact.runbook
- owner_role: role.devops
- owner_binding: Registered release owner accountable for deployment and rollback readiness.
- allowed_transitions:
  - lifecycle.operations
  - lifecycle.incident

### lifecycle.operations
- rule_id: rule.lifecycle.operations
- purpose: Operate the released system, maintain service health, collect feedback, and govern steady-state changes.
- entry_criteria:
  - artifact.release-checklist
  - artifact.runbook
- exit_criteria:
  - transition_to_incident: trigger_event_recorded
  - transition_to_deprecation: artifact.deprecation-plan
- owner_role: role.devops
- owner_binding: Registered service owner accountable for production operation.
- allowed_transitions:
  - lifecycle.incident
  - lifecycle.deprecation

### lifecycle.incident
- rule_id: rule.lifecycle.incident
- purpose: Record, triage, mitigate, and communicate production failures or high-severity operational risks.
- entry_criteria:
  - trigger_event_recorded
- exit_criteria:
  - artifact.incident-report
- owner_role: role.devops
- owner_binding: Registered incident commander or service owner.
- allowed_transitions:
  - lifecycle.postmortem

### lifecycle.postmortem
- rule_id: rule.lifecycle.postmortem
- purpose: Determine root cause, corrective actions, preventive actions, and governance updates required after an incident.
- entry_criteria:
  - artifact.incident-report
- exit_criteria:
  - artifact.postmortem
- owner_role: role.em
- owner_binding: Registered corrective-action owner accountable for follow-through.
- allowed_transitions:
  - lifecycle.operations

### lifecycle.deprecation
- rule_id: rule.lifecycle.deprecation
- purpose: Remove or retire a system, capability, workflow, or commitment under an approved retirement plan.
- entry_criteria:
  - artifact.release-checklist
  - artifact.runbook
- exit_criteria:
  - artifact.deprecation-plan
- owner_role: role.pm
- owner_binding: Registered business or portfolio owner accountable for retirement.
- allowed_transitions: []

### rule.lifecycle.transition-equality
- statement: Exit criteria of stage N MUST equal entry criteria of stage N+1 for the main forward lifecycle chain only.
- main_chain:
  - lifecycle.bootstrap -> lifecycle.discovery
  - lifecycle.discovery -> lifecycle.scoping
  - lifecycle.scoping -> lifecycle.design
  - lifecycle.design -> lifecycle.planning
  - lifecycle.planning -> lifecycle.implementation
  - lifecycle.implementation -> lifecycle.testing
  - lifecycle.testing -> lifecycle.release
  - lifecycle.release -> lifecycle.operations
- boundary_sets:
  - lifecycle.bootstrap.exit == lifecycle.discovery.entry == [artifact.role-registry, artifact.artifact-registry, artifact.sla-registry, artifact.event-log, artifact.decision-log]
  - lifecycle.discovery.exit == lifecycle.scoping.entry == [artifact.prd]
  - lifecycle.scoping.exit == lifecycle.design.entry == [artifact.prd, artifact.prd.scope_constraints != empty, artifact.prd.threshold_classification != empty, artifact.prd.customer_impact != empty, artifact.prd.execution_mode != empty, artifact.prd.status == active]
  - lifecycle.design.exit == lifecycle.planning.entry == [artifact.tech-design-doc, artifact.architecture-diagram]
  - lifecycle.planning.exit == lifecycle.implementation.entry == [artifact.task, artifact.test-plan, artifact.sprint-plan]
  - lifecycle.implementation.exit == lifecycle.testing.entry == [artifact.test-results, artifact.runbook]
  - lifecycle.testing.exit == lifecycle.release.entry == [artifact.test-results, artifact.test-results.status == active, artifact.security-review, artifact.security-review.status == active, artifact.test-results.release_recommendation == accepted]
  - lifecycle.release.exit == lifecycle.operations.entry == [artifact.release-checklist, artifact.runbook]
- exclusion_note: lifecycle.operations exits are event-driven or deprecation-driven and are not part of the main forward boundary equality rule.
- enforcement: Any main-chain transition with a non-matching boundary set is INVALID.

### rule.lifecycle.boundary-evaluation-mode
- statement: Lifecycle boundaries are satisfied only when both artifact presence and declared field or state conditions are met.
- requirement: Artifact presence alone is insufficient when a lifecycle block defines additional field or state requirements.
- enforcement: Any gate evaluator that validates only artifact existence for a mixed boundary is INVALID.

### rule.lifecycle.small-change-mode
- statement: Small-change mode MAY collapse discovery, scoping, and design review depth for low-risk work but MUST NOT bypass non-overridable invariants.
- entry_condition:
  - linked_decision_log.classification == THRESHOLD_MINOR
  - customer_impact == low
  - security_privacy_impact == none
  - rollback_feasible == true
  - effort_estimate < 3-developer-days
- required_retained_artifacts:
  - artifact.prd
  - artifact.task
  - artifact.test-plan
  - artifact.runbook
- enforcement: Any small-change execution missing retained artifacts or violating entry conditions is INVALID.

### rule.lifecycle.scoping-readiness
- statement: Scoping is valid only when the governing PRD is field-complete and ready for design intake.
- required_fields:
  - scope_constraints
  - threshold_classification
  - customer_impact
  - execution_mode
- enforcement: A PRD entering design without all required scoping fields is INVALID.

### rule.lifecycle.testing-readiness
- statement: Testing is complete only when shared evidence has been validated and formal release disposition exists.
- required_exit_state:
  - artifact.test-results.status == active
  - artifact.security-review.status == active
  - artifact.test-results.release_recommendation == accepted
- enforcement: Any release gate lacking all testing exit conditions is INVALID.

### rule.lifecycle.operations-transition-policy
- statement: Operations is a steady-state lifecycle with transition-specific exits.
- transition_requirements:
  - lifecycle.operations -> lifecycle.incident requires trigger_event_recorded
  - lifecycle.operations -> lifecycle.deprecation requires artifact.deprecation-plan
- enforcement: Any operations exit lacking the matching transition requirement is INVALID.

### rule.lifecycle.people-ops-model
- statement: Hiring and onboarding use a sibling governed lifecycle model that inherits this playbook's artifact, decision, status, event, versioning, and override rules.
- people_ops_stages:
  - people-lifecycle.hiring
  - people-lifecycle.onboarding
- mappings:
  - artifact.hiring-scorecard -> people-lifecycle.hiring
  - artifact.onboarding-checklist -> people-lifecycle.onboarding
- enforcement: Delivery lifecycle stages MUST NOT be used to validate hiring or onboarding artifacts.

### rule.lifecycle.people-ops-deferred-definition
- statement: The core declares the people-lifecycle namespace and shared primitive rules, while full people-lifecycle stage definitions are completed in a people-ops extension playbook.
- requirement: Until that extension exists, only lifecycle mapping and artifact ownership are governed here, not full people-stage transition rules.
- enforcement: Treating the people lifecycle as fully defined within this core is INVALID.

## 3. Artifact Contracts

### artifact.prd
- rule_id: rule.artifact.prd
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - problem_statement
  - target_outcomes
  - scope_constraints
  - threshold_classification
  - execution_mode
  - effort_estimate
  - acceptance_criteria
  - customer_impact
- lifecycle_stage_created: lifecycle.discovery
- lifecycle_stage_consumed:
  - lifecycle.scoping
  - lifecycle.design
- validation_rules:
  - artifact_id MUST equal artifact.prd.
  - owner_role MUST be a registered product or domain owner role.
  - source_lifecycle_stage MUST equal lifecycle.discovery.
  - target_lifecycle_stage MUST include lifecycle.scoping.
  - target_lifecycle_stage MUST include lifecycle.design.
  - status MUST be governed by `rule.status.state-machine`.
  - version MUST be governed by `rule.version.artifact`.
  - linked_decision_ids MUST contain at least one decision ID.
  - problem_statement, target_outcomes, scope_constraints, threshold_classification, execution_mode, effort_estimate, acceptance_criteria, and customer_impact MUST be non-empty.

### artifact.tech-design-doc
- rule_id: rule.artifact.tech-design-doc
- owner_role: role.backend
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - owner_role MUST be a registered engineering domain owner role.
  - source_lifecycle_stage MUST equal lifecycle.design.
  - target_lifecycle_stage MUST include lifecycle.planning.
  - target_lifecycle_stage MUST include lifecycle.implementation.
  - requirements_reference MUST reference artifact.prd.
  - interface_definitions, dependency_map, and risk_register MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.architecture-diagram
- rule_id: rule.artifact.architecture-diagram
- owner_role: role.backend
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - system_boundaries
  - component_nodes
  - data_flows
  - trust_boundaries
  - embedding_mode
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.planning
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.architecture-diagram.
  - owner_role MUST be a registered engineering domain owner role.
  - source_lifecycle_stage MUST equal lifecycle.design.
  - target_lifecycle_stage MUST include lifecycle.planning.
  - target_lifecycle_stage MUST include lifecycle.implementation.
  - component_nodes, data_flows, and trust_boundaries MUST be non-empty.
  - embedding_mode MUST be one of [standalone, embedded-in-tech-design-doc].
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.task
- rule_id: rule.artifact.task
- owner_role: role.em
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - referenced_versions
- lifecycle_stage_created: lifecycle.planning
- lifecycle_stage_consumed:
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.task.
  - owner_role MUST be a registered delivery owner role.
  - source_lifecycle_stage MUST equal lifecycle.planning.
  - target_lifecycle_stage MUST equal lifecycle.implementation.
  - assignee_role MUST be a registered role ID.
  - parent_design_references MUST reference artifact.tech-design-doc or artifact.architecture-diagram.
  - referenced_versions MUST identify specific accepted upstream versions.
  - definition_of_done and dependency_references MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.test-plan
- rule_id: rule.artifact.test-plan
- owner_role: role.qa
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - referenced_versions
- lifecycle_stage_created: lifecycle.planning
- lifecycle_stage_consumed:
  - lifecycle.implementation
  - lifecycle.testing
- validation_rules:
  - artifact_id MUST equal artifact.test-plan.
  - owner_role MUST be a registered quality owner role.
  - source_lifecycle_stage MUST equal lifecycle.planning.
  - target_lifecycle_stage MUST include lifecycle.implementation.
  - target_lifecycle_stage MUST include lifecycle.testing.
  - test_scenarios, pass_fail_criteria, and observability_requirements MUST be non-empty.
  - referenced_versions MUST identify specific accepted upstream versions.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.test-results
- rule_id: rule.artifact.test-results
- owner_role: role.qa
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - contributor_signoff_roles
  - referenced_versions
- lifecycle_stage_created: lifecycle.implementation
- lifecycle_stage_consumed:
  - lifecycle.testing
  - lifecycle.release
- validation_rules:
  - artifact_id MUST equal artifact.test-results.
  - owner_role MUST be a registered quality owner role.
  - source_lifecycle_stage MUST equal lifecycle.implementation.
  - target_lifecycle_stage MUST include lifecycle.testing.
  - target_lifecycle_stage MUST include lifecycle.release.
  - test_plan_reference MUST reference artifact.test-plan.
  - execution_summary, evidence_links, and contributor_signoff_roles MUST be non-empty.
  - referenced_versions MUST identify specific accepted upstream versions.
  - release_recommendation MUST be one of [accepted, rejected, needs-revision].
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.release-checklist
- rule_id: rule.artifact.release-checklist
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - referenced_versions
- conditional_required_fields:
  - security_review_reference when change_class includes security-sensitive, vendor-integrating, or externally-exposing-data
- lifecycle_stage_created: lifecycle.release
- lifecycle_stage_consumed:
  - lifecycle.operations
  - lifecycle.deprecation
- validation_rules:
  - artifact_id MUST equal artifact.release-checklist.
  - owner_role MUST be a registered release owner role.
  - source_lifecycle_stage MUST equal lifecycle.release.
  - target_lifecycle_stage MUST include lifecycle.operations.
  - rollback_steps, approval_evidence, and operational_readiness MUST be non-empty.
  - referenced_versions MUST identify specific accepted upstream versions.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.runbook
- rule_id: rule.artifact.runbook
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - referenced_versions
- lifecycle_stage_created: lifecycle.implementation
- lifecycle_stage_consumed:
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
  - lifecycle.incident
- validation_rules:
  - artifact_id MUST equal artifact.runbook.
  - owner_role MUST be a registered service or operations owner role.
  - source_lifecycle_stage MUST equal lifecycle.implementation.
  - target_lifecycle_stage MUST include lifecycle.testing.
  - target_lifecycle_stage MUST include lifecycle.release.
  - target_lifecycle_stage MUST include lifecycle.operations.
  - target_lifecycle_stage MUST include lifecycle.incident.
  - observability_endpoints, rollback_reference, and recovery_steps MUST be non-empty.
  - referenced_versions MUST identify specific accepted upstream versions.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.incident-report
- rule_id: rule.artifact.incident-report
- owner_role: role.devops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - remediation_due_by
- lifecycle_stage_created: lifecycle.incident
- lifecycle_stage_consumed:
  - lifecycle.postmortem
- validation_rules:
  - artifact_id MUST equal artifact.incident-report.
  - owner_role MUST be a registered service or incident owner role.
  - source_lifecycle_stage MUST equal lifecycle.incident.
  - target_lifecycle_stage MUST equal lifecycle.postmortem.
  - severity MUST be one of [severity.p1, severity.p2, severity.p3].
  - timeline, mitigation_actions, and remediation_due_by MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.postmortem
- rule_id: rule.artifact.postmortem
- owner_role: role.em
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - owner_role MUST be a registered corrective-action owner role.
  - source_lifecycle_stage MUST equal lifecycle.postmortem.
  - target_lifecycle_stage MUST equal lifecycle.operations.
  - incident_reference MUST reference artifact.incident-report.
  - root_cause, corrective_actions, and preventive_actions MUST be non-empty.
  - closure_owner_role MUST be a registered role ID.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.deprecation-plan
- rule_id: rule.artifact.deprecation-plan
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
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
  - owner_role MUST be a registered portfolio owner role.
  - source_lifecycle_stage MUST equal lifecycle.deprecation.
  - target_lifecycle_stage MUST equal lifecycle.deprecation.
  - retirement_scope, retirement_date, migration_path, dependency_impact, and approval_evidence MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.okr
- rule_id: rule.artifact.okr
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - objective
  - key_results
  - measurement_window
  - owning_domain
- lifecycle_stage_created: lifecycle.discovery
- lifecycle_stage_consumed:
  - lifecycle.scoping
  - lifecycle.planning
- validation_rules:
  - artifact_id MUST equal artifact.okr.
  - objective, key_results, measurement_window, and owning_domain MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.roadmap
- rule_id: rule.artifact.roadmap
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - planning_horizon
  - prioritized_items
  - dependency_summary
  - review_cycle
- lifecycle_stage_created: lifecycle.scoping
- lifecycle_stage_consumed:
  - lifecycle.discovery
  - lifecycle.planning
  - lifecycle.deprecation
- validation_rules:
  - artifact_id MUST equal artifact.roadmap.
  - planning_horizon, prioritized_items, dependency_summary, and review_cycle MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.sprint-plan
- rule_id: rule.artifact.sprint-plan
- owner_role: role.em
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - execution_order
  - dependency_graph
  - capacity_allocation
  - milestone_dates
  - referenced_versions
- lifecycle_stage_created: lifecycle.planning
- lifecycle_stage_consumed:
  - lifecycle.implementation
- validation_rules:
  - artifact_id MUST equal artifact.sprint-plan.
  - execution_order, dependency_graph, capacity_allocation, milestone_dates, and referenced_versions MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.support-escalation
- rule_id: rule.artifact.support-escalation
- owner_role: role.support-intake
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - issue_summary
  - customer_impact
  - urgency_class
  - source_channel
- optional_fields:
  - supporting_roles
- lifecycle_stage_created: lifecycle.operations
- lifecycle_stage_consumed:
  - lifecycle.discovery
  - lifecycle.scoping
- validation_rules:
  - artifact_id MUST equal artifact.support-escalation.
  - owner_role MUST be a locked core or registered support or service intake role.
  - issue_summary, customer_impact, urgency_class, and source_channel MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.analytics-spec
- rule_id: rule.artifact.analytics-spec
- owner_role: role.data
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - metric_definitions
  - event_schema
  - dashboard_targets
  - validation_queries
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.testing
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.analytics-spec.
  - metric_definitions, event_schema, dashboard_targets, and validation_queries MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.dashboard
- rule_id: rule.artifact.dashboard
- owner_role: role.data
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - dashboard_url
  - source_metrics
  - refresh_interval
  - alert_thresholds
- lifecycle_stage_created: lifecycle.release
- lifecycle_stage_consumed:
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.dashboard.
  - dashboard_url, source_metrics, refresh_interval, and alert_thresholds MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.hiring-scorecard
- rule_id: rule.artifact.hiring-scorecard
- owner_role: role.people-ops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - role_scope
  - evaluation_dimensions
  - rubric
  - decision_criteria
- lifecycle_stage_created: people-lifecycle.hiring
- lifecycle_stage_consumed:
  - people-lifecycle.hiring
- validation_rules:
  - artifact_id MUST equal artifact.hiring-scorecard.
  - owner_role MUST be a locked core or registered people-operations role.
  - role_scope, evaluation_dimensions, rubric, and decision_criteria MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.onboarding-checklist
- rule_id: rule.artifact.onboarding-checklist
- owner_role: role.people-ops
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - role_scope
  - required_access
  - training_steps
  - completion_criteria
- lifecycle_stage_created: people-lifecycle.onboarding
- lifecycle_stage_consumed:
  - people-lifecycle.onboarding
- validation_rules:
  - artifact_id MUST equal artifact.onboarding-checklist.
  - owner_role MUST be a locked core or registered people-operations role.
  - role_scope, required_access, training_steps, and completion_criteria MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.vendor-eval
- rule_id: rule.artifact.vendor-eval
- owner_role: role.vendor-owner
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - vendor_name
  - risk_summary
  - compliance_findings
  - approval_recommendation
  - security_owner_role
  - requesting_owner_role
- optional_fields:
  - supporting_roles
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.release
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.vendor-eval.
  - owner_role MUST be a locked core or registered vendor requesting owner role.
  - security_owner_role MUST be a registered security role.
  - requesting_owner_role MUST be a registered role.
  - vendor_name, risk_summary, compliance_findings, and approval_recommendation MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.security-review
- rule_id: rule.artifact.security-review
- owner_role: role.security
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - scope_reference
  - threat_findings
  - remediation_requirements
  - release_disposition
- optional_fields:
  - supporting_roles
- lifecycle_stage_created: lifecycle.testing
- lifecycle_stage_consumed:
  - lifecycle.release
- validation_rules:
  - artifact_id MUST equal artifact.security-review.
  - scope_reference, threat_findings, remediation_requirements, and release_disposition MUST be non-empty.
  - release_disposition MUST be one of [accepted, rejected, needs-revision].
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.privacy-impact
- rule_id: rule.artifact.privacy-impact
- owner_role: role.security
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - data_categories
  - processing_purpose
  - retention_policy
  - mitigation_requirements
- lifecycle_stage_created: lifecycle.design
- lifecycle_stage_consumed:
  - lifecycle.release
- validation_rules:
  - artifact_id MUST equal artifact.privacy-impact.
  - data_categories, processing_purpose, retention_policy, and mitigation_requirements MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.budget-request
- rule_id: rule.artifact.budget-request
- owner_role: role.pm
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - requested_amount
  - spending_reason
  - funding_window
  - approval_status
- lifecycle_stage_created: lifecycle.scoping
- lifecycle_stage_consumed:
  - lifecycle.planning
  - lifecycle.release
- validation_rules:
  - artifact_id MUST equal artifact.budget-request.
  - requested_amount, spending_reason, funding_window, and approval_status MUST be non-empty.
  - linked_decision_ids MUST contain at least one decision ID.

### artifact.decision-log
- rule_id: rule.artifact.decision-log
- owner_role: role.decision-steward
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - decision_id
  - driver_role
  - context
  - decision
  - rationale
  - approver_role
  - contributing_roles
  - linked_artifact_ids
  - linked_lifecycle_stage_ids
  - linked_versions
  - linked_event_ids
  - timestamp
  - threshold_classification
  - outcome
- optional_fields:
  - predecessor_decision_ids
  - supporting_roles
- lifecycle_stage_created: lifecycle.bootstrap
- lifecycle_stage_consumed:
  - lifecycle.bootstrap
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
- validation_rules:
  - artifact_id MUST equal artifact.decision-log.
  - owner_role MUST be a locked core or registered decision record steward role.
  - driver_role MUST be a registered role ID.
  - decision_id MUST be unique.
  - approver_role MUST be a registered role ID.
  - linked_artifact_ids MUST be non-empty.
  - linked_lifecycle_stage_ids MUST be non-empty.
  - linked_versions MUST be non-empty.
  - linked_event_ids MUST resolve against artifact.event-log.
  - predecessor_decision_ids MAY be empty for first-instance decisions and MUST resolve when present.
  - threshold_classification MUST be one of [THRESHOLD_MINOR, THRESHOLD_MAJOR].
  - outcome MUST be one of [accepted, rejected, needs-revision].

### artifact.event-log
- rule_id: rule.artifact.event-log
- owner_role: role.decision-steward
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - source_lifecycle_stage
  - target_lifecycle_stage
  - linked_decision_ids
  - event_records
- lifecycle_stage_created: lifecycle.bootstrap
- lifecycle_stage_consumed:
  - lifecycle.bootstrap
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
- validation_rules:
  - artifact_id MUST equal artifact.event-log.
  - owner_role MUST be a locked core or registered decision record steward role.
  - event_records MUST be append-only.
  - each event record MUST satisfy rule.event.definition.
  - linked_decision_ids MUST contain at least one decision ID for governed event batches.

### artifact.role-registry
- rule_id: rule.artifact.role-registry
- owner_role: role.cto
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - linked_decision_ids
  - registered_ids
  - registration_records
- lifecycle_stage_created: lifecycle.bootstrap
- lifecycle_stage_consumed:
  - lifecycle.bootstrap
  - lifecycle.discovery
  - lifecycle.planning
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.role-registry.
  - registered_ids and registration_records MUST be non-empty.
  - each registration record MUST satisfy rule.role.registration-record-format.
  - each registration record MUST satisfy rule.registry.record-status.
  - each registration record MUST satisfy rule.registry.record-versioning.

### artifact.artifact-registry
- rule_id: rule.artifact.artifact-registry
- owner_role: role.cto
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - linked_decision_ids
  - registered_ids
  - registration_records
- lifecycle_stage_created: lifecycle.bootstrap
- lifecycle_stage_consumed:
  - lifecycle.bootstrap
  - lifecycle.discovery
  - lifecycle.planning
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.artifact-registry.
  - registered_ids and registration_records MUST be non-empty.
  - each registration record MUST satisfy rule.artifact.registration-record-format.
  - each registration record MUST satisfy rule.registry.record-status.
  - each registration record MUST satisfy rule.registry.record-versioning.

### artifact.sla-registry
- rule_id: rule.artifact.sla-registry
- owner_role: role.cto
- required_fields:
  - artifact_id
  - owner_role
  - status
  - version
  - previous_version_id
  - created_at
  - updated_at
  - linked_decision_ids
  - registered_ids
  - registration_records
- lifecycle_stage_created: lifecycle.bootstrap
- lifecycle_stage_consumed:
  - lifecycle.bootstrap
  - lifecycle.discovery
  - lifecycle.planning
  - lifecycle.testing
  - lifecycle.release
  - lifecycle.operations
- validation_rules:
  - artifact_id MUST equal artifact.sla-registry.
  - registered_ids and registration_records MUST be non-empty.
  - each registration record MUST satisfy rule.sla.registration-record-format.
  - each registration record MUST satisfy rule.registry.record-status.
  - each registration record MUST satisfy rule.registry.record-versioning.

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

### sla-profile.document-review
- rule_id: rule.sla-profile.document-review
- parent_class: sla.standard
- response_time: 4-business-hours
- completion_time: 3-business-days
- escalation_trigger: completion_deadline_at_risk
- applicable_events:
  - event.artifact.created
  - event.artifact.updated

### sla-profile.support-triage
- rule_id: rule.sla-profile.support-triage
- parent_class: sla.standard
- response_time: 4-business-hours
- completion_time: 1-business-day
- escalation_trigger: response_time_exceeded
- applicable_events:
  - event.artifact.created
  - event.lifecycle.entered

### sla-profile.release-risk
- rule_id: rule.sla-profile.release-risk
- parent_class: sla.expedited
- response_time: 2-hours
- completion_time: 1-business-day
- escalation_trigger: immediate_on_severity_classification
- applicable_events:
  - event.violation.detected
  - event.sla.breached

### rule.sla.registration-policy
- statement: Base SLA classes are locked; derived SLA profiles MAY exist only through governed registration in `artifact.sla-registry`.
- locked_base_classes:
  - sla.standard
  - sla.expedited
  - sla.emergency
- allowed_registered_profiles:
  - sla-profile.document-review
  - sla-profile.support-triage
  - sla-profile.release-risk
- enforcement: Any unregistered SLA class or profile is INVALID.

### severity.p1
- rule_id: rule.severity.p1
- response_binding: sla.emergency
- definition: Production outage, data exposure, severe customer harm, or inability to restore service safely without immediate action.

### severity.p2
- rule_id: rule.severity.p2
- response_binding: sla.expedited
- definition: Major degradation, significant customer impact, or release risk requiring urgent coordinated remediation.

### severity.p3
- rule_id: rule.severity.p3
- response_binding: sla.standard
- definition: Contained issue with limited impact and no immediate threat to customer safety, security, or contractual commitments.

### rule.sla.trigger-mapping
- statement: SLA application MUST be event-bound.
- mappings:
  - event.incident.severity-p1 -> sla.emergency
  - event.incident.severity-p2 -> sla.expedited
  - event.incident.severity-p3 -> sla.standard
  - event.document-review.requested -> sla-profile.document-review
  - event.support-triage.requested -> sla-profile.support-triage
  - event.release-risk.detected -> sla-profile.release-risk
- enforcement: Any event without a registered SLA binding is INVALID.

## 5. Decision Framework

### rule.decision.daci-model
- decision_model: DACI
- archetypes:
  - driver: registered domain owner
  - approver: registered accountable approver
  - contributor: affected registered roles
  - informed: interested registered roles
- clarification: The driver coordinates the decision process; the approver holds final authority.
- canonical_approver_model: A decision MUST have exactly one approver role per decision record.
- evidence_artifact: artifact.decision-log

### rule.decision.threshold-minor
- threshold_id: THRESHOLD_MINOR
- default_definition: Single-domain change, reversible within one release window, no material security/privacy exposure, no external commitment change, and no net-new vendor dependency.
- requirement: Changes classified at or below THRESHOLD_MINOR MUST be approved by the designated accountable approver for the impacted workflow or artifact.
- enforcement: A decision record without threshold classification is INVALID.

### rule.decision.threshold-major
- threshold_id: THRESHOLD_MAJOR
- default_definition: Multi-domain change, externally visible contract change, material security/privacy exposure, irreversible data/process change, new vendor dependency, material spend commitment, or organization-wide workflow change.
- requirement: Changes classified at or above THRESHOLD_MAJOR MUST be approved by the registered accountable function head; governance changes MUST be approved by role.cto.
- enforcement: A major decision without accountable approver evidence is INVALID.

### rule.threshold.must-be-defined
- statement: Threshold meanings MUST be operationally defined.
- requirement: Any inheriting playbook overriding threshold semantics MUST define a decision classification matrix and replacement rule.
- enforcement: Undefined threshold semantics are INVALID.

### rule.decision.conflict-escalation
- statement: Equal-level disagreement MUST escalate to the next accountable approver in the registered escalation chain.
- protocol:
  - domain-level disagreement -> accountable approver
  - accountable approver disagreement -> registered function head
  - cross-function deadlock -> role.cto
- enforcement: A blocked decision without escalation record in `artifact.decision-log` is INVALID.

## 6. Handoff Contract System

### Discovery -> Scoping
- rule_id: rule.handoff.discovery-to-scoping
- required_artifacts:
  - artifact.prd
  - artifact.decision-log
- validation_conditions:
  - artifact.prd MUST be active or under-review.
  - artifact.prd MUST include owner_role, customer_impact, and linked_decision_ids.
  - gate record MUST include from_stage lifecycle.discovery and to_stage lifecycle.scoping.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered scoping owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - non-canonical or unregistered owner_role
  - empty acceptance_criteria in artifact.prd

### Scoping -> Design
- rule_id: rule.handoff.scoping-to-design
- required_artifacts:
  - artifact.prd
  - artifact.decision-log
- validation_conditions:
  - artifact.prd MUST include scope_constraints, threshold_classification, customer_impact, and execution_mode.
  - gate record MUST include from_stage lifecycle.scoping and to_stage lifecycle.design.
  - budget-relevant work MUST reference artifact.budget-request.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered design owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - missing threshold classification
  - missing budget-request for budget-relevant work

### Design -> Implementation
- rule_id: rule.handoff.design-to-implementation
- required_artifacts:
  - artifact.tech-design-doc
  - artifact.architecture-diagram
  - artifact.task
  - artifact.test-plan
  - artifact.sprint-plan
- validation_conditions:
  - artifact.task MUST reference artifact.tech-design-doc or artifact.architecture-diagram.
  - artifact.test-plan MUST include observability_requirements.
  - artifact.sprint-plan MUST include execution_order and dependency_graph.
  - gate record MUST include from_stage lifecycle.design and to_stage lifecycle.implementation.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered implementation owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty definition_of_done
  - empty observability_requirements
  - empty dependency_graph

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
  - accepting_role MUST be a registered quality owner role.
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
  - artifact.security-review
- validation_conditions:
  - artifact.test-results.release_recommendation MUST equal accepted.
  - artifact.runbook MUST include escalation_steps and recovery_steps.
  - artifact.security-review.release_disposition MUST equal accepted.
  - gate record MUST include from_stage lifecycle.testing and to_stage lifecycle.release.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered release owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - release_recommendation not accepted
  - release_disposition not accepted
  - empty recovery_steps

### Release -> Operations
- rule_id: rule.handoff.release-to-operations
- required_artifacts:
  - artifact.release-checklist
  - artifact.runbook
  - artifact.dashboard
- validation_conditions:
  - artifact.release-checklist MUST include rollback_steps and operational_readiness.
  - artifact.runbook MUST be active.
  - artifact.dashboard MUST define source_metrics and alert_thresholds.
  - gate record MUST include from_stage lifecycle.release and to_stage lifecycle.operations.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered service owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty rollback_steps
  - empty operational_readiness
  - missing dashboard

### Incident -> Postmortem
- rule_id: rule.handoff.incident-to-postmortem
- required_artifacts:
  - artifact.incident-report
  - artifact.decision-log
- validation_conditions:
  - artifact.incident-report MUST include severity, impact_window, mitigation_actions, timeline, and remediation_due_by.
  - gate record MUST include from_stage lifecycle.incident and to_stage lifecycle.postmortem.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered corrective-action owner role.
- rejection_conditions:
  - missing artifact reference
  - missing acceptance_state
  - empty timeline
  - empty mitigation_actions

### Support/Feedback -> Roadmap
- rule_id: rule.handoff.support-feedback-to-roadmap
- required_artifacts:
  - artifact.support-escalation
  - artifact.decision-log
  - artifact.roadmap
- validation_conditions:
  - artifact.support-escalation MUST include issue_summary, customer_impact, urgency_class, and source_channel.
  - artifact.decision-log MUST classify the intake outcome.
  - gate record MUST identify target planning cycle in artifact.roadmap.
- acceptance_signal:
  - acceptance_state MUST be one of [accepted, rejected, needs-revision].
  - accepting_role MUST be a registered roadmap owner role.
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
- enforcement_depth: minimal artifact set with explicit permitted embedding and small-change collapse
- explicit_deltas:
  - artifact.architecture-diagram MAY use embedding_mode embedded-in-tech-design-doc
  - Discovery, Scoping, and Design MAY execute in small-change mode when `rule.lifecycle.small-change-mode` is satisfied
  - role substitution MUST be declared in artifact.decision-log when a specialized registered role is absent
  - spike-mode MAY be approved by decision-log override for THRESHOLD_MINOR exploratory work that does not produce accepted implementation tasks
  - artifact.security-review remains REQUIRED for security-sensitive work
- required_reviews:
  - owner_role review for all lifecycle exits
  - accountable approver review for THRESHOLD_MAJOR
- audit_requirements:
  - gate records REQUIRED
  - artifact.decision-log REQUIRED
  - full artifact history OPTIONAL except for release, incident, override, and security-sensitive changes

### activation.16-75
- rule_id: rule.activation.16-75
- team_size: 16-75
- enforcement_depth: full lifecycle gate enforcement
- explicit_deltas:
  - artifact.architecture-diagram SHOULD be standalone except for THRESHOLD_MINOR work
  - artifact.security-review REQUIRED for release-bound work with external exposure, vendor integration, or sensitive data
  - artifact.analytics-spec and artifact.dashboard REQUIRED for metrics-bearing product changes
- required_reviews:
  - owner_role review for all lifecycle exits
  - accountable approver review for THRESHOLD_MINOR and THRESHOLD_MAJOR
  - independent quality signoff for release
- audit_requirements:
  - artifact history REQUIRED
  - gate records REQUIRED
  - decision records REQUIRED for all lifecycle transitions

### activation.76-200
- rule_id: rule.activation.76-200
- team_size: 76-200
- enforcement_depth: strict full-system enforcement
- explicit_deltas:
  - embedded architecture diagrams are INVALID
  - independent security review REQUIRED for any public, regulated, or vendor-integrated release
  - privacy-impact, vendor-eval, budget-request, and dashboard artifacts MUST be present when change class requires them
  - override actions REQUIRE separate governance approval evidence
- required_reviews:
  - owner_role review
  - accountable approver review
  - independent reviewer evidence for release, incident, postmortem, override, and major vendor/security decisions
- audit_requirements:
  - complete audit trail REQUIRED for all artifacts, decisions, transitions, and overrides

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
- requirement: Every decision MUST be recorded in artifact.decision-log and linked to affected artifacts.
- enforcement: Any undocumented decision is INVALID.

### rule.quality.system-validity
- statement: This playbook system is valid only when all mandatory structures are present and consistent.
- measurable_conditions:
  - frontmatter schema completeness == 100 percent
  - lifecycle stage coverage == all locked lifecycle IDs in rule.machine.canonical-ids-only
  - artifact contract coverage == all locked and registered artifact IDs in rule.machine.canonical-ids-only
  - handoff contract coverage == all mandatory handoff rule IDs
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
- description: Work is advanced without a referenced artifact or decision record.
- detection_signal: Lifecycle transition record exists with no artifact_reference and no artifact.decision-log reference.
- consequence: Transition is INVALID and MUST be rolled back to the prior valid stage.

### anti-pattern.02-ownerless-artifact
- description: An artifact exists with no owner_role or more than one owner_role.
- detection_signal: Artifact metadata contains zero or multiple owner_role fields.
- consequence: Artifact is INVALID and MUST be rejected from all handoffs.

### anti-pattern.03-unlogged-decision
- description: A decision changes scope, design, release, or operations state without a decision artifact.
- detection_signal: State change exists with no linked_decision_ids or no artifact.decision-log entry.
- consequence: Decision is INVALID and dependent artifacts MUST be marked needs-revision.

### anti-pattern.04-ungated-transition
- description: A lifecycle stage changes without explicit acceptance_state.
- detection_signal: from_stage and to_stage are recorded without acceptance_state.
- consequence: Transition is INVALID and MUST NOT be recognized by downstream stages.

### anti-pattern.05-noncanonical-id
- description: A lifecycle, artifact, role, SLA, severity, profile, event, or people-lifecycle value uses an unregistered string.
- detection_signal: ID value is not present in the locked or registered canonical set.
- consequence: Record is INVALID and MUST be corrected before processing.

### anti-pattern.06-design-without-traceability
- description: A design artifact does not reference its originating requirements artifact.
- detection_signal: artifact.tech-design-doc or artifact.architecture-diagram has no reference to artifact.prd.
- consequence: Design handoff is INVALID and MUST be rejected.

### anti-pattern.07-testing-without-observability
- description: Testing proceeds without observability requirements or endpoints.
- detection_signal: artifact.test-plan.observability_requirements is empty or artifact.runbook.observability_endpoints is empty.
- consequence: Testing handoff is INVALID and release eligibility is denied.

### anti-pattern.08-release-without-rollback
- description: Release approval is attempted without rollback documentation.
- detection_signal: artifact.release-checklist.rollback_steps is empty or artifact.runbook.rollback_reference is empty.
- consequence: Release is INVALID and MUST be blocked.

### anti-pattern.09-closed-incident-without-postmortem
- description: An incident is marked resolved without a postmortem artifact.
- detection_signal: artifact.incident-report status indicates closure and no artifact.postmortem reference exists.
- consequence: Incident closure is INVALID and operational compliance is failed.

### anti-pattern.10-handoff-without-acceptance
- description: A handoff package is delivered without explicit receiver acceptance state.
- detection_signal: handoff record missing acceptance_state or accepting_role.
- consequence: Handoff is INVALID and ownership does not transfer.

### anti-pattern.11-override-without-replacement
- description: A downstream playbook claims an override but does not define a replacement rule.
- detection_signal: override record cites a rule ID but lacks replacement_rule.
- consequence: Override is INVALID and parent rule remains in force.

### anti-pattern.12-conflicting-boundary-set
- description: Adjacent lifecycle stages define non-matching entry and exit artifact sets on the main chain.
- detection_signal: Lifecycle boundary comparison returns inequality.
- consequence: Lifecycle specification is INVALID and must be corrected before inheritance.

## 11. Inheritance & Override Rules

### rule.inheritance.required
- statement: All playbooks MUST inherit this document.
- requirement: Every non-core playbook frontmatter MUST reference `core-operating-playbook` in `governing_parent`.
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

### rule.machine.frontmatter-schema
- statement: This playbook defines the authoritative frontmatter schema for all playbooks.
- required_keys:
  - id
  - type
  - owner_role
  - governing_parent
  - related_playbooks
  - related_lifecycle_namespaces
  - lifecycle_stages
  - primary_input_artifacts
  - primary_output_artifacts
  - trigger_events
  - sla_class
  - review_cadence
  - status
- enforcement: Any playbook missing a required key is INVALID.

### rule.machine.fixed-section-structure
- statement: The core playbook section order is fixed and parsable; inheriting playbooks MUST preserve order for any sections they implement.
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
  - 15. System Enforcement Engine
  - 16. Global Status State Machine
  - 17. System Dependency Graph
  - 18. Decision Record System
  - 19. System Event Model
  - 20. Concurrency Model
  - 21. Versioning Model
- enforcement: The core playbook MUST use the full required order; descendants that implement any of these sections MUST preserve the same relative order.

### rule.machine.canonical-ids-only
- statement: All IDs MUST use locked core strings or registered extension strings only.
- locked_core_ids:
  - lifecycle:
      - lifecycle.bootstrap
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
  - people_lifecycle:
      - people-lifecycle.hiring
      - people-lifecycle.onboarding
  - artifact:
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
      - artifact.okr
      - artifact.roadmap
      - artifact.sprint-plan
      - artifact.support-escalation
      - artifact.analytics-spec
      - artifact.dashboard
      - artifact.hiring-scorecard
      - artifact.onboarding-checklist
      - artifact.vendor-eval
      - artifact.security-review
      - artifact.privacy-impact
      - artifact.budget-request
      - artifact.decision-log
      - artifact.event-log
      - artifact.role-registry
      - artifact.artifact-registry
      - artifact.sla-registry
  - role:
      - role.cto
      - role.pm
      - role.em
      - role.backend
      - role.frontend
      - role.qa
      - role.devops
      - role.security
      - role.design
      - role.data
      - role.support-intake
      - role.people-ops
      - role.vendor-owner
      - role.decision-steward
  - sla:
      - sla.standard
      - sla.expedited
      - sla.emergency
  - sla_profile:
      - sla-profile.document-review
      - sla-profile.support-triage
      - sla-profile.release-risk
  - severity:
      - severity.p1
      - severity.p2
      - severity.p3
  - event:
      - event.artifact.created
      - event.artifact.updated
      - event.artifact.accepted
      - event.artifact.rejected
      - event.lifecycle.entered
      - event.lifecycle.exited
      - event.violation.detected
      - event.sla.breached
      - event.decision.recorded
      - event.incident.severity-p1
      - event.incident.severity-p2
      - event.incident.severity-p3
      - event.document-review.requested
      - event.support-triage.requested
      - event.release-risk.detected
- registered_extension_patterns:
  - role.<extension>
  - artifact.<extension>
  - sla-profile.<extension>
  - event.<extension>
- enforcement: Any non-canonical or unregistered ID is INVALID.

### rule.machine.traceable-relationships
- statement: Lifecycle-to-artifact and role-to-artifact relationships MUST be explicit and traceable.
- required_relationship_fields:
  - source_lifecycle_stage
  - target_lifecycle_stage
  - owner_role
  - linked_decision_ids
  - version
- enforcement: Any artifact contract or artifact instance without explicit relationships is INVALID.

### rule.machine.explicit-cross-references
- statement: Cross-references MUST be explicit and MUST NOT be implied.
- requirement: Every dependency, predecessor, successor, parent artifact, decision reference, version reference, and event reference MUST be declared by canonical ID.
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
  - propose change with artifact.decision-log entry
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
- requirement: Any breaking governance change MUST include migration guidance in the approving decision record.
- enforcement: A breaking change without major version increment is INVALID.

### rule.role.registration-required
- statement: New role IDs MAY be used only after governed registration in artifact.role-registry.
- requirement: Any new `role.<extension>` MUST be added through a governance update with owner semantics, escalation chain, absorbed-by fallback, and registration record.
- enforcement: Unregistered role IDs are INVALID.

### rule.role.registration-record-format
- required_fields:
  - registered_id
  - display_name
  - owner_archetype
  - registration_decision_id
  - effective_version
  - status
  - escalation_target
  - absorbed_by_when_absent
- storage_artifact: artifact.role-registry
- enforcement: A role used without a matching active role-registry record is INVALID.

### rule.artifact.registration-required
- statement: New artifact IDs MAY be used only after governed registration in artifact.artifact-registry.
- requirement: Any new `artifact.<extension>` MUST define owner_role, required_fields, lifecycle mapping, dependency edges, and registration record.
- enforcement: Unregistered artifact IDs are INVALID.

### rule.artifact.registration-record-format
- required_fields:
  - registered_id
  - display_name
  - owner_archetype
  - registration_decision_id
  - effective_version
  - status
  - required_fields
  - lifecycle_mapping
  - dependency_edges
- storage_artifact: artifact.artifact-registry
- enforcement: An artifact used without a matching active artifact-registry record is INVALID.

### rule.governance.sla-registration
- statement: New SLA profiles MAY be used only after governed registration in artifact.sla-registry.
- requirement: Any new `sla-profile.<extension>` MUST declare parent base class, response target, completion target, escalation trigger, event binding, and registration record.
- enforcement: Unregistered SLA profiles are INVALID.

### rule.sla.registration-record-format
- required_fields:
  - registered_id
  - display_name
  - owner_archetype
  - registration_decision_id
  - effective_version
  - status
  - parent_class
  - response_time
  - completion_time
  - event_binding
- storage_artifact: artifact.sla-registry
- enforcement: An SLA profile used without a matching active sla-registry record is INVALID.

### rule.registry.record-status
- valid_states:
  - draft
  - active
  - deprecated
- referenceability_rule: Only active registration records MAY be referenced by inherited playbooks.
- enforcement: Referencing a draft or deprecated registration record is INVALID.

### rule.registry.record-versioning
- statement: Registration records inherit semantic versioning.
- requirement: Breaking changes to registration records MUST use a major version increment.
- enforcement: A registration update with mismatched version semantics is INVALID.

### rule.registry.lookup-validity
- statement: A referenced role, artifact, or SLA profile is valid only if the registry artifact is active, the registration record is active, and the requested effective version is compatible.
- enforcement: Any lookup failing one of these conditions is INVALID.

### rule.governance.superseded-rule-deprecation
- statement: A superseded rule MUST remain referenceable until all inheriting playbooks migrate.
- requirement: superseded rules MUST be marked deprecated, reference successor rule IDs, and retain historical traceability.
- enforcement: Removing a referenced rule without successor mapping is INVALID.

## 14. Validation Criteria

### rule.validation.lifecycle-connected
- requirement: All lifecycle stages MUST be connected through allowed transitions.
- validation_method: Verify every locked lifecycle ID appears in Section 2 and participates in at least one incoming or outgoing transition except lifecycle.bootstrap incoming and lifecycle.deprecation outgoing.

### rule.validation.artifacts-referenced
- requirement: All locked and registered artifact IDs MUST be referenced by at least one lifecycle stage, one artifact contract, and one handoff, quality, governance, dependency, event, or version rule.
- validation_method: Verify full artifact coverage across Sections 2, 3, 6, 9, 13, 17, 19, and 21.

### rule.validation.handoffs-defined
- requirement: All eight mandatory handoffs MUST be defined exactly once.
- validation_method: Verify one and only one block exists for each required handoff contract.

### rule.validation.no-conflicting-ids
- requirement: No duplicate or conflicting canonical IDs MAY exist.
- validation_method: Verify uniqueness across lifecycle IDs, people-lifecycle IDs, artifact IDs, role IDs, SLA IDs, severity IDs, event IDs, rule IDs, and anti-pattern IDs.

### rule.validation.yaml-schema-consistent
- requirement: YAML frontmatter MUST contain all required schema keys exactly once and MUST use canonical or explicitly allowed root values.
- validation_method: Verify keys [id, type, owner_role, governing_parent, related_playbooks, related_lifecycle_namespaces, lifecycle_stages, primary_input_artifacts, primary_output_artifacts, trigger_events, sla_class, review_cadence, status] are present exactly once.

## 15. System Enforcement Engine

### rule.enforcement.global
- detection: Any violation of a rule, invariant, validation, lifecycle gate, handoff contract, status transition, dependency edge, event schema, concurrency rule, or version rule.
- actions:
  - block lifecycle transition
  - mark affected artifacts as needs-revision
  - emit enforcement event
  - require remediation artifact or decision record
- escalation:
  - first violation -> owner_role remediation
  - repeated violation -> registered accountable approver
  - systemic violation -> role.cto
- recovery_condition:
  - violation resolved
  - validation passes
  - new acceptance_state recorded

### rule.enforcement.break-glass-incident
- statement: Incident intake MUST NOT be blocked by missing pre-incident documentation.
- requirement: Missing runbook or missing prior operational artifacts during lifecycle.incident MUST generate remediation requirements instead of blocking incident entry.
- remediation_sla: sla.expedited.completion_time
- remediation_deadline: Remediation artifacts MUST be created within 2-business-days of incident closure.
- enforcement: Incident intake is VALID on trigger_event_recorded; remediation artifacts become REQUIRED before incident closure.

## 16. Global Status State Machine

### rule.status.state-machine
- valid_states:
  - draft
  - under-review
  - active
  - deprecated
  - needs-revision
- transitions:
  - draft -> under-review
  - under-review -> active
  - under-review -> needs-revision
  - needs-revision -> under-review
  - active -> deprecated
- transition_requirements:
  - draft -> under-review requires linked_decision_ids
  - under-review -> active requires acceptance_state == accepted
  - under-review -> needs-revision requires rejection or remediation evidence
  - needs-revision -> under-review requires remediation evidence
  - active -> deprecated requires successor reference or artifact.deprecation-plan
- transition_owner:
  - draft -> under-review: artifact.owner_role
  - under-review -> active: receiving owner_role or accountable approver
  - under-review -> needs-revision: receiving owner_role or enforcement engine
  - needs-revision -> under-review: artifact.owner_role
  - active -> deprecated: accountable owner_role with decision record
- enforcement: Any invalid status transition is rejected.

### rule.acceptance.rejection
- statement: Rejected artifacts MUST re-enter review through a new versioned remediation cycle.
- required_after_rejection:
  - rejection_reason
  - version increment
  - updated acceptance state
  - remediation evidence
- enforcement: A rejected artifact that re-enters review without these fields is INVALID.

## 17. System Dependency Graph

### rule.graph.artifact-dependencies
- edges:
  - artifact.role-registry -> artifact.decision-log
  - artifact.artifact-registry -> artifact.decision-log
  - artifact.sla-registry -> artifact.decision-log
  - artifact.event-log -> artifact.decision-log
  - artifact.okr -> artifact.prd
  - artifact.support-escalation -> artifact.prd
  - artifact.prd -> artifact.roadmap
  - artifact.prd -> artifact.tech-design-doc
  - artifact.tech-design-doc -> artifact.architecture-diagram
  - artifact.tech-design-doc -> artifact.task
  - artifact.architecture-diagram -> artifact.task
  - artifact.task -> artifact.test-plan
  - artifact.task -> artifact.sprint-plan
  - artifact.test-plan -> artifact.test-results
  - artifact.sprint-plan -> artifact.test-results
  - artifact.analytics-spec -> artifact.dashboard
  - artifact.test-results -> artifact.security-review
  - artifact.security-review -> artifact.release-checklist
  - artifact.runbook -> artifact.release-checklist
  - artifact.release-checklist -> artifact.dashboard
  - artifact.release-checklist -> artifact.incident-report
  - artifact.incident-report -> artifact.postmortem
  - artifact.release-checklist -> artifact.deprecation-plan
- enforcement: Missing required dependency edges are INVALID.

### rule.graph.lifecycle-artifact-mapping
- mappings:
  - lifecycle.bootstrap -> [artifact.role-registry, artifact.artifact-registry, artifact.sla-registry, artifact.event-log, artifact.decision-log]
  - lifecycle.discovery -> [artifact.okr, artifact.support-escalation, artifact.prd, artifact.decision-log, artifact.event-log]
  - lifecycle.scoping -> [artifact.prd, artifact.roadmap, artifact.budget-request, artifact.decision-log, artifact.event-log]
  - lifecycle.design -> [artifact.tech-design-doc, artifact.architecture-diagram, artifact.analytics-spec, artifact.vendor-eval, artifact.privacy-impact, artifact.decision-log, artifact.event-log]
  - lifecycle.planning -> [artifact.task, artifact.test-plan, artifact.sprint-plan, artifact.decision-log, artifact.event-log]
  - lifecycle.implementation -> [artifact.test-results, artifact.runbook, artifact.decision-log, artifact.event-log]
  - lifecycle.testing -> [artifact.test-results, artifact.security-review, artifact.decision-log, artifact.event-log]
  - lifecycle.release -> [artifact.release-checklist, artifact.dashboard, artifact.decision-log, artifact.event-log]
  - lifecycle.operations -> [artifact.support-escalation, artifact.dashboard, artifact.decision-log, artifact.event-log, artifact.role-registry, artifact.artifact-registry, artifact.sla-registry]
  - lifecycle.incident -> [artifact.incident-report, artifact.decision-log, artifact.event-log]
  - lifecycle.postmortem -> [artifact.postmortem, artifact.decision-log, artifact.event-log]
  - lifecycle.deprecation -> [artifact.deprecation-plan, artifact.decision-log, artifact.event-log]
  - people-lifecycle.hiring -> [artifact.hiring-scorecard, artifact.decision-log, artifact.event-log]
  - people-lifecycle.onboarding -> [artifact.onboarding-checklist, artifact.decision-log, artifact.event-log]
- enforcement: Any artifact used outside its declared lifecycle mapping without an override record is INVALID.

### rule.graph.handoff-artifact-mapping
- mappings:
  - rule.handoff.discovery-to-scoping -> [artifact.prd, artifact.decision-log]
  - rule.handoff.scoping-to-design -> [artifact.prd, artifact.decision-log, artifact.budget-request]
  - rule.handoff.design-to-implementation -> [artifact.tech-design-doc, artifact.architecture-diagram, artifact.task, artifact.test-plan, artifact.sprint-plan]
  - rule.handoff.implementation-to-testing -> [artifact.test-results, artifact.runbook]
  - rule.handoff.testing-to-release -> [artifact.test-results, artifact.runbook, artifact.security-review]
  - rule.handoff.release-to-operations -> [artifact.release-checklist, artifact.runbook, artifact.dashboard]
  - rule.handoff.incident-to-postmortem -> [artifact.incident-report, artifact.decision-log]
  - rule.handoff.support-feedback-to-roadmap -> [artifact.support-escalation, artifact.decision-log, artifact.roadmap]
- enforcement: A handoff using artifacts outside its registered mapping is INVALID.

## 18. Decision Record System

### rule.decision.record-system
- decision_artifact: artifact.decision-log
- required_use_cases:
  - lifecycle gate approvals
  - threshold classification
  - support feedback intake
  - override approval
  - SLA profile registration
  - artifact and role registration
  - incident command assignment
- enforcement: Any decision-bearing event without artifact.decision-log is INVALID.

### rule.decision.record-linkage
- statement: Every artifact or lifecycle transition referencing a decision MUST link to `artifact.decision-log.decision_id`.
- requirement:
  - linked_decision_ids on non-decision artifacts MUST resolve to existing artifact.decision-log records
  - predecessor_decision_ids on artifact.decision-log MUST resolve when present
  - linked_artifact_ids, linked_lifecycle_stage_ids, linked_versions, and linked_event_ids in the decision record MUST reciprocally reference the governed objects
- enforcement: Any non-resolving or one-way decision linkage is INVALID.

## 19. System Event Model

### rule.event.definition
- event_types:
  - event.artifact.created
  - event.artifact.updated
  - event.artifact.accepted
  - event.artifact.rejected
  - event.lifecycle.entered
  - event.lifecycle.exited
  - event.violation.detected
  - event.sla.breached
  - event.decision.recorded
  - event.incident.severity-p1
  - event.incident.severity-p2
  - event.incident.severity-p3
  - event.document-review.requested
  - event.support-triage.requested
  - event.release-risk.detected
- required_event_record_fields:
  - event_id
  - event_type
  - timestamp
  - actor_role
  - artifact_id
  - artifact_version
  - lifecycle_stage_id
  - decision_id
  - metadata
- enforcement: Any event without the required schema is INVALID.

### rule.event.triggers
- required_triggers:
  - artifact acceptance emits event.lifecycle.entered when the next stage boundary is satisfied
  - artifact rejection emits event.violation.detected or remediation-needed metadata
  - SLA breach emits event.sla.breached
  - incident trigger emits event.lifecycle.entered for lifecycle.incident
  - decision creation emits event.decision.recorded
- enforcement: Any required trigger omitted by an event-bearing state change is INVALID.

### rule.event.subscriptions
- subscription_dimensions:
  - ownership
  - dependency_edge
  - lifecycle_stage
  - governance_scope
- enforcement: Event consumers MUST subscribe based on declared ownership or dependency relationship; implicit subscriptions are INVALID.

## 20. Concurrency Model

### rule.concurrency.execution
- statement: Parallel work is allowed only when explicit dependencies, versions, and invariants are satisfied.
- permissions:
  - multiple artifacts MAY be active within the same lifecycle stage
  - multiple lifecycle stages MAY be active concurrently only when dependency edges are satisfied
  - downstream artifacts MUST NOT advance to accepted if required upstream dependencies are unresolved or invalid
- enforcement: Any concurrent advancement violating dependency or version rules is INVALID.

### rule.concurrency.constraints
- constraints:
  - artifact.tech-design-doc and artifact.architecture-diagram MUST be accepted before dependent implementation tasks are accepted
  - artifact.test-plan MAY begin during planning before all tasks exist
  - artifact.analytics-spec, artifact.security-review, and artifact.privacy-impact MAY proceed in parallel where declared dependency edges permit
  - release MUST NOT begin until all required concurrent artifacts resolve to accepted
- enforcement: A concurrent artifact accepted before its required constraints are satisfied is INVALID.

### rule.concurrency.conflict-resolution
- statement: Conflicting concurrent updates require explicit reconciliation.
- requirements:
  - artifact version increment
  - reconciliation decision in artifact.decision-log
  - new acceptance state for impacted dependents when a breaking update occurs
- enforcement: Conflicting updates without reconciliation are INVALID.

## 21. Versioning Model

### rule.version.artifact
- statement: Every artifact MUST be versioned.
- required_fields:
  - version
  - previous_version_id
- version_format: semantic-versioning
- enforcement: Any artifact missing required version fields is INVALID.

### rule.version.increment
- patch: non-meaning-changing correction or metadata update
- minor: additive non-breaking change
- major: breaking change requiring downstream revalidation
- enforcement: Any version increment that does not match the impact of the artifact change is INVALID.

### rule.version.enforcement
- statement: Dependent artifacts MUST reference specific upstream versions.
- requirements:
  - dependent artifacts MUST reference specific upstream versions
  - major version updates MUST invalidate dependent acceptance until revalidated
  - rejected artifacts MUST produce a new version before re-entering review
- enforcement: Any dependent artifact lacking specific version references or relying on invalidated major versions is INVALID.
