# Sub-Agent API Contracts for Project Aether

## 1. Introduction
This document provides the initial, high-level API contract definitions for the gRPC services that enable communication between Aether's specialized sub-agents. These definitions use Protocol Buffers (protobuf) syntax.

These contracts are not exhaustive but represent the core interactions required for the system to function. They will be expanded upon during the implementation phase.

---

## 2. Core Services

### 2.1 PlanningService
The `PlanningService` is the central orchestration point. It is used by the Planning/Reasoning Engine to manage the overall project lifecycle.

```protobuf
syntax = "proto3";

package aether.services;

// Manages the overall project plan and lifecycle.
service PlanningService {
  // Initiates a new project from high-level requirements.
  rpc StartProject(StartProjectRequest) returns (StartProjectResponse);

  // Retrieves the status of a task.
  rpc GetTaskStatus(GetTaskStatusRequest) returns (Task);

  // Reports the completion or failure of a task.
  rpc UpdateTaskStatus(UpdateTaskStatusRequest) returns (UpdateTaskStatusResponse);
}

message StartProjectRequest {
  string project_name = 1;
  string high_level_requirements = 2;
}

message StartProjectResponse {
  string project_id = 1;
  string initial_plan_id = 2;
}

message GetTaskStatusRequest {
  string task_id = 1;
}

message UpdateTaskStatusRequest {
  string task_id = 1;
  TaskStatus status = 2;
  string details = 3; // e.g., error message, link to artifact
}

message UpdateTaskStatusResponse {
  bool acknowledged = 1;
}

message Task {
  string task_id = 1;
  string description = 2;
  TaskStatus status = 3;
  repeated string dependencies = 4; // Other task_ids
  string assigned_agent = 5; // e.g., "ArchitectAgent"
}

enum TaskStatus {
  PENDING = 0;
  IN_PROGRESS = 1;
  COMPLETED = 2;
  FAILED = 3;
}
```

### 2.2 MemoryService
The `MemoryService` is used by all agents to interact with the persistent vector database.

```protobuf
syntax = "proto3";

package aether.services;

// Provides access to the long-term persistent memory (vector DB).
service MemoryService {
  // Adds or updates an artifact in the knowledge base.
  rpc StoreArtifact(StoreArtifactRequest) returns (StoreArtifactResponse);

  // Retrieves relevant artifacts based on a semantic query.
  rpc RetrieveContext(RetrieveContextRequest) returns (RetrieveContextResponse);
}

message Artifact {
  string id = 1; // e.g., file_path, commit_hash
  ArtifactType type = 2;
  string content = 3;
  map<string, string> metadata = 4;
}

enum ArtifactType {
  SOURCE_CODE = 0;
  DOCUMENTATION = 1;
  TEST_RESULT = 2;
  ERROR_LOG = 3;
  ARCHITECTURE_DIAGRAM = 4;
}

message StoreArtifactRequest {
  Artifact artifact = 1;
}

message StoreArtifactResponse {
  string artifact_id = 1;
  bool success = 2;
}

message RetrieveContextRequest {
  string query_text = 1;
  int32 top_k = 2; // Number of results to return
  map<string, string> metadata_filter = 3;
}

message RetrieveContextResponse {
  repeated Artifact relevant_artifacts = 1;
}
```

---

## 3. Agent-Specific Services

### 3.1 ArchitectureService
The `ArchitectAgent` uses this service to define the technical plan.

```protobuf
syntax = "proto3";

package aether.services;

// Used by the ArchitectAgent to design the system.
service ArchitectureService {
  // Decomposes a high-level requirement into a detailed technical plan.
  rpc CreateTechnicalPlan(CreateTechnicalPlanRequest) returns (CreateTechnicalPlanResponse);
}

message CreateTechnicalPlanRequest {
  string project_id = 1;
  string requirement_description = 2;
}

message CreateTechnicalPlanResponse {
  string plan_id = 1;
  repeated Task implementation_tasks = 2; // Tasks for CoderAgent, TestAgent, etc.
}
```

### 3.2 CodingService
The `CoderAgent` implements features based on tasks.

```protobuf
syntax = "proto3";

package aether.services;

// Used by the CoderAgent to implement code.
service CodingService {
  // Implements a specific function or module.
  rpc ImplementCode(ImplementCodeRequest) returns (ImplementCodeResponse);
}

message ImplementCodeRequest {
  string task_id = 1;
  string specifications = 2; // Detailed specs from ArchitectAgent
  string relevant_context = 3; // Context from MemoryService
}

message ImplementCodeResponse {
  string commit_hash = 1;
  repeated string files_changed = 2;
}
```

### 3.3 TestingService
The `TestAgent` verifies the implemented code.

```protobuf
syntax = "proto3";

package aether.services;

// Used by the TestAgent to run tests.
service TestingService {
  // Generates and runs tests for a given piece of code.
  rpc RunTests(RunTestsRequest) returns (RunTestsResponse);
}

message RunTestsRequest {
  string commit_hash = 1;
  repeated string files_to_test = 2;
}

message TestResult {
  string test_name = 1;
  bool passed = 2;
  string message = 3;
}

message RunTestsResponse {
  bool all_passed = 1;
  repeated TestResult results = 2;
  double coverage_percentage = 3;
}
```
