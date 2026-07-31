# Required Requirement Inputs

- **Project purpose:** A fitness web application that provides science-based exercise plans and diet plans, supports customization, and lets users track their progress.
- **Primary users / actors:** UNKNOWN (notes imply an end user who follows plans and tracks progress; no roles named)
- **Core workflows:** Obtain/follow an exercise plan; customize a plan; obtain/follow a diet plan; track progress
- **Business objects / data entities:** Exercise plan, diet plan, progress record
- **External integrations:** UNKNOWN
- **Authentication / roles:** UNKNOWN
- **Regulatory or privacy constraints:** UNKNOWN

# Functional Requirements

### Delivery Channel

- **FR-1.1** The system MUST be delivered as a web application accessible through a web browser.

### Exercise Plans

- **FR-2.1** The system MUST provide users with exercise plans.
- **FR-2.2** The system MUST allow a user to customize an exercise plan, and MUST persist the customized plan for that user.
- **FR-2.3** Each exercise plan the system provides MUST be science-based, meaning it MUST display the supporting evidence or source for its content. Acceptance: opening any plan shows at least one attributed source. Content criteria for "science-based" are TO BE DECIDED.

### Diet Plans

- **FR-3.1** The system MUST provide users with diet plans.
- **FR-3.2** The system MUST allow a user to customize a diet plan, and MUST persist the customized plan for that user.
- **FR-3.3** Each diet plan the system provides MUST be science-based, meaning it MUST display the supporting evidence or source for its content. Acceptance: opening any plan shows at least one attributed source. Content criteria for "science-based" are TO BE DECIDED.

### Progress Tracking

- **FR-4.1** The system MUST allow a user to record progress entries.
- **FR-4.2** The system MUST display a user's recorded progress back to that user.
- **FR-4.3** The system MUST scope plans, customizations, and progress records to the user who created them, and MUST NOT expose one user's records to another user.

# Open Questions

- **OQ-1** Is an account required, and what authentication method and roles apply? Without this, FR-4.3 scoping and all persistence requirements are untestable.
- **OQ-2** What progress metrics are tracked (weight, body measurements, workout completion, lifted volume, calories, photos), and at what granularity/frequency?
- **OQ-3** What defines "science-based" — citation to peer-reviewed sources, adherence to a named guideline, or expert review? Who verifies it?
- **OQ-4** What does "custom" cover — user-authored plans from scratch, editing system-provided plans, or generated plans from user inputs (goals, equipment, dietary restrictions)?
- **OQ-5** Where do plans come from (pre-authored library, admin-curated catalog, generated), and can users create or share plans?
- **OQ-6** What diet plan detail is required (meals, recipes, macro/calorie targets, food logging)? Is food logging part of progress tracking?
- **OQ-7** Are health/nutrition data privacy obligations (e.g. HIPAA, GDPR) in scope, and is a medical disclaimer required?
- **OQ-8** Are there any external integrations (wearables, nutrition databases, payments) or is the system fully self-contained?
- **OQ-9** Is the product free, paid, or subscription-based?
- **OQ-10** What platforms and devices must the web app support (mobile browsers, offline use, accessibility targets)?
