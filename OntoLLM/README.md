# OntoLLM Ontologies

This directory contains domain-specific ontologies used in the OntoLLM framework for enhancing conversational AI systems with structured knowledge representation. These ontologies enable precise context tracking, prevent conversational drift, and support accurate information retrieval across multi-domain industrial scenarios.

## Overview

The OntoLLM framework uses JSON-based ontologies to define:
- **Nodes**: Entity types with their properties and data types
- **Relation Trees**: Semantic relationships between entities using graph notation

Each ontology includes special "Question" nodes that bridge structured knowledge graphs with unstructured document content through unique vector reference identifiers.

## Ontology Files

### 1. Motor Ontology (`motor.json`)

**Domain**: Equipment Maintenance and Motor Management  
**Use Case**: Industrial motor maintenance, installation procedures, and technical documentation

#### Entities:
- **Motor**: Individual motor instances (`MotorName: Text`)
- **Model**: Motor model specifications (`ModelName: Text`)
- **Location**: Installation locations (`LocationName: Text`)
- **Phase**: Electrical phase configurations (`PhaseName: Text`)
- **Manual**: Technical documentation (`ManualName: Text`)
- **Question**: Bridge nodes for unstructured content (`ChunkVectorRefID: Numeric`, `ChunkQuestion: Text`)

#### Key Relationships:

- Motor → hasModel → Model 
- Motor → hasLocation → Location 
- Model → hasPhase → Phase 
- Model → hasManual → Manual 
- Manual → hasQuestions → Question 
- Question → related → Question


#### Example Scenario:
Distinguishing between motor models "X100" and "Y200" to prevent cross-contamination in maintenance procedures.

---

### 2. Employee Ontology (`employee.json`)

**Domain**: Human Resources and Organizational Management  
**Use Case**: Employee information management, role assignments, and policy compliance

#### Entities:
- **Employee**: Individual employees (`EmployeeName: Text`)
- **Role**: Job roles and positions (`RoleName: Text`)
- **Type**: Employee categories (`TypeName: Text`)
- **Department**: Organizational departments (`DepartmentName: Text`)
- **Policy**: Departmental policies (`PolicyName: Text`)
- **Question**: Bridge nodes for policy documents (`ChunkVectorRefID: Numeric`, `ChunkQuestion: Text`)

#### Key Relationships:

- Employee → hasRole → Role 
- Employee → hasType → Type 
- Employee → hasDepartment → Department 
- Department → hasPolicy → Policy 
- Policy → hasQuestion → Question 
- Question → related → Question


#### Example Scenario:
Managing different leave policies for Engineering vs HR departments to ensure accurate policy information retrieval.

---

### 3. Exploration Plan Ontology (`explorationplan.json`)

**Domain**: Oil & Gas Operations and Offshore Drilling  
**Use Case**: BSEE exploration plan management, well operations, and regulatory compliance

#### Entities:
- **Well**: Drilling well instances (`WellName: Text`)
- **Block**: Offshore blocks (`BlockNumber: Text`)
- **Area**: Geographic areas (`AreaName: Text`)
- **Lease**: Drilling leases (`LeaseNumber: Text`)
- **Region**: Regulatory regions (`RegionName: Text`)
- **ExplorationPlan**: BSEE documentation (`ControlNumber: Text`, `Type: Text`, `DateReceived: Date`)
- **Question**: Bridge nodes for plan documents (`ChunkVectorRefID: Numeric`, `ChunkQuestion: Text`)

#### Key Relationships:

Well → hasBlock → Block 
Well → hasArea → Area 
Well → hasLease → Lease 
Well → hasRegion → Region 
Block → hasExplorationPlan → ExplorationPlan 
ExplorationPlan → hasQuestions → Question 
Block → hasArea → Area 
Block → hasLease → Lease 
Area → hasRegion → Region 
Question → related → Question


#### Example Scenario:
Managing exploration plans across different offshore blocks in Gulf of Mexico vs Pacific regions.

---

### 4. Right-of-Way Ontology (`rightofway.json`)

**Domain**: Oil & Gas Infrastructure and Pipeline Management  
**Use Case**: BSEE Right-of-Way grant management, pipeline segments, and regulatory documentation

#### Entities:
- **Region**: Regulatory regions (`RegionName: Text`)
- **Segment**: Pipeline/infrastructure segments (`SegmentNumber: Text`)
- **Type**: ROW grant types (`TypeName: Text`)
- **ROW**: Right-of-Way documents (`ROWNumber: Text`, `DocDate: Date`, `ImportedDate: Date`)
- **Question**: Bridge nodes for ROW documents (`ChunkVectorRefID: Numeric`, `ChunkQuestion: Text`)

#### Key Relationships:

Region → hasSegment → Segment 
Region → hasType → Type 
Segment → hasROW → ROW 
ROW → hasQuestions → Question 
Question → related → Question


