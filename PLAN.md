# C# ACP SDK vs Python Official SDK Gap Analysis

## Context

The goal is to align the C# ACP SDK's functionality with the Python official SDK (based on schema v0.12.2). Below is a comprehensive comparison of the two SDKs, listing all missing feature modules.

---

## 1. Missing Session Lifecycle Methods

The Python SDK supports full session management. C# is missing:

| Feature | JSON-RPC Method | C# Status |
|---------|----------------|-----------|
| List sessions | `session/list` | Missing |
| Fork session | `session/fork` | Missing |
| Resume session | `session/resume` | Missing |
| Close session | `session/close` | Missing |

New additions needed:
- `ListSessionsRequest/Response`, `ForkSessionRequest/Response`, `ResumeSessionRequest/Response`, `CloseSessionRequest/Response` (Schema types)
- 4 corresponding constants in `AgentMethods`
- Corresponding methods in `IAcpAgent` interface
- Corresponding handlers registered in `AgentSideConnection`

---

## 2. Missing Session Config Option Feature

| Feature | JSON-RPC Method | C# Status |
|---------|----------------|-----------|
| Set config option | `session/set_config_option` | Missing |

New additions needed:
- `SetSessionConfigOptionBooleanRequest`, `SetSessionConfigOptionSelectRequest`, `SetSessionConfigOptionResponse` (Schema types)
- `SessionConfigOptionBoolean`, `SessionConfigOptionSelect`, `SessionConfigSelectOption`, `SessionConfigSelectGroup` etc.
- `AgentMethods.SessionSetConfigOption` constant
- `IAcpAgent.SetSessionConfigOptionAsync` method

---

## 3. Missing SessionUpdate Subtypes

Python SDK's `SessionUpdate` has 11 subtypes, C# only has 8. Missing:

| SessionUpdate Subtype | C# Status |
|----------------------|-----------|
| `config_option_update` | Missing |
| `session_info_update` | Missing |
| `usage_update` | Missing |

New additions needed:
- `ConfigOptionUpdateSessionUpdate` (with `config_options` field)
- `SessionInfoUpdateSessionUpdate` (with `title`, `updated_at` fields)
- `UsageUpdateSessionUpdate` (with `size`, `used`, `cost` fields)
- Supporting types: `Cost`, `Usage`

---

## 4. Missing Elicitation Feature

**Entirely new module** — completely absent from C#:

JSON-RPC methods:
- `elicitation/create` (Agent -> Client request)
- `elicitation/complete` (Client -> Agent notification)

New Schema types needed (~15+):
- `CreateFormElicitationRequest`, `CreateUrlElicitationRequest`
- `ElicitationSchema`, `ElicitationStringPropertySchema`, `ElicitationNumberPropertySchema`, `ElicitationIntegerPropertySchema`, `ElicitationBooleanPropertySchema`, `ElicitationMultiSelectPropertySchema`
- `AcceptElicitationResponse`, `DeclineElicitationResponse`, `CancelElicitationResponse`
- `CompleteElicitationNotification`
- `ElicitationFormMode`, `ElicitationUrlMode` (polymorphic)
- Various Scope types

New additions needed:
- `ClientMethods` constants (`ElicitationCreate`, `ElicitationComplete`)
- `ElicitationCreateAsync`, `CompleteElicitationAsync` methods in `IAcpClient`
- `CompleteElicitationAsync` method in `IAcpAgent` (notification direction)
- `Elicitation` field in `ClientCapabilities`

---

## 5. Missing NES (Next Edit Suggestion) Feature

**Entirely new module** — completely absent from C#:

JSON-RPC methods:
- `nes/start`, `nes/suggest`, `nes/accept`, `nes/reject`, `nes/close` (Client -> Agent)
- `document/didOpen`, `document/didChange`, `document/didClose`, `document/didSave`, `document/didFocus` (Client -> Agent)

New Schema types needed (~25+):
- All NES suggestion types: `NesEditSuggestion`, `NesJumpSuggestion`, `NesRenameSuggestion`, `NesSearchAndReplaceSuggestion` and their Variants
- NES context types: `NesTextEdit`, `NesDiagnostic`, `NesEditHistoryEntry`, `NesExcerpt`, `NesRelatedSnippet`, `NesOpenFile`, `NesRecentFile`, `NesUserAction`, `NesRepository`
- NES request/response: `StartNesRequest/Response`, `SuggestNesRequest/Response`, `CloseNesRequest/Response`
- NES notifications: `AcceptNesNotification`, `RejectNesNotification`
- Document change types: `DidOpenDocumentNotification`, `DidChangeDocumentNotification`, `DidCloseDocumentNotification`, `DidSaveDocumentNotification`, `DidFocusDocumentNotification`
- NES-related Capabilities types (~10)

New additions needed:
- `AgentMethods` constants (10: 5 NES + 5 Document)
- Corresponding methods in `IAcpAgent` interface
- `Nes` field in `ClientCapabilities`
- `Nes` field in `AgentCapabilities`

---

## 6. Missing Provider Management Feature

**Entirely new module** — completely absent from C#:

JSON-RPC methods:
- `providers/list`, `providers/set`, `providers/disable` (Client -> Agent)

New Schema types needed:
- `ListProvidersRequest/Response`, `SetProvidersRequest/Response`, `DisableProvidersRequest/Response`
- `ProviderInfo`, `ProviderCurrentConfig`

New additions needed:
- `AgentMethods` constants
- Corresponding methods in `IAcpAgent` interface
- `Providers` field in `AgentCapabilities`

---

## 7. Missing Logout Feature

| Feature | JSON-RPC Method | C# Status |
|---------|----------------|-----------|
| Logout | `logout` | Missing |

New additions needed:
- `LogoutRequest/Response`
- `AgentMethods.Logout` constant
- `IAcpAgent.LogoutAsync` method
- `AgentAuthCapabilities`, `LogoutCapabilities` types

---

## 8. Missing AuthMethod Subtypes

Python SDK's AuthMethod is polymorphic (3 types). C# only has a single `AuthMethod` type:

| AuthMethod Type | C# Status |
|----------------|-----------|
| `AuthMethodAgent` (base) | Present (current AuthMethod) |
| `EnvVarAuthMethod` (type="env_var") | Missing |
| `TerminalAuthMethod` (type="terminal") | Missing |
| `AuthEnvVar` | Missing |

Need to refactor `AuthMethod` into a polymorphic type using `type` as the discriminator.

---

## 9. Missing Base Types

| Type | Purpose | C# Status |
|------|---------|-----------|
| `Position` | Zero-based row/column position | Missing |
| `Range` | Text range | Missing |
| `Cost` | Cost (amount + currency) | Missing |
| `Usage` | Token usage statistics | Missing |
| `WorkspaceFolder` | Workspace folder | Missing |
| `SessionInfo` | Session metadata | Missing |
| `EnumOption` | Enum option | Missing |

---

## 10. Missing Capability Types

C# capabilities are much simpler than Python's:

| Capability | Fields | C# Status |
|-----------|--------|-----------|
| `AuthCapabilities` | terminal | Missing |
| `AgentAuthCapabilities` | logout | Missing |
| `ElicitationCapabilities` | form, url | Missing |
| `SessionCapabilities` | additional_directories, close, fork, list, resume | Missing |
| `ProvidersCapabilities` | (marker) | Missing |
| `ClientNesCapabilities` | jump, rename, search_and_replace | Missing |
| `NesCapabilities` | context, events | Missing |
| NES-related sub-Capabilities (~8) | Various editor context capabilities | Missing |
| `PositionEncodings` | position_encodings | Missing |

Need to refactor `ClientCapabilities` and `AgentCapabilities` to include these fields.

---

## 11. Missing PromptRequest Field

| Field | C# Status |
|-------|-----------|
| `message_id` | Missing |

---

## 12. Missing NewSessionRequest / LoadSessionRequest Fields

| Field | C# Status |
|-------|-----------|
| `additional_directories` | Missing |

---

## 13. ContentBlock annotations Field

Python SDK has an `annotations` field on all ContentBlock subtypes. C# is missing this field.

---

## 14. ToolCallContent Polymorphic Discriminator Difference

| Python | C# |
|--------|-----|
| discriminator = `type` ("content"/"diff"/"terminal") | Based on property existence |
| `FileEditToolCallContent` (type="diff") | `DiffToolCallContent` (determined by path/newText) |

Python uses an explicit `type` field, C# relies on property existence. Recommend unifying to use `type` discriminator.

---

## 15. RequestPermissionOutcome Difference

| Python | C# |
|--------|-----|
| Uses `outcome` field ("selected"/"cancelled") | Uses `optionId` property existence |
| `AllowedOutcome`, `DeniedOutcome` | `SelectedRequestPermissionOutcome`, `CancelledRequestPermissionOutcome` |

Recommend aligning to Python's `outcome` discriminator approach.

---

## 16. Missing ToolCall Field

| Field | C# Status |
|-------|-----------|
| `title` (required) | Present |

In Python SDK's `ToolCall.content`, the `title` of `ToolCallStart` is required — C# also has it as required, so this is consistent. However, `RequestPermissionRequest.tool_call` is `ToolCallUpdate` type (with `tool_call_id`) in Python, but `object` type in C#. Recommend changing to `ToolCallUpdateSessionUpdate` type.

---

## TODO List

| # | Issue | Done | Tested |
|---|-------|------|--------|
| 1 | Missing Session lifecycle methods (list/fork/resume/close) | [√] | [x] |
| 2 | Missing Session Config Option feature | [ ] | [ ] |
| 3 | Missing SessionUpdate subtypes (config_option_update/session_info_update/usage_update) | [ ] | [ ] |
| 4 | Missing Elicitation feature | [ ] | [ ] |
| 5 | Missing NES (Next Edit Suggestion) feature | [ ] | [ ] |
| 6 | Missing Provider management feature | [ ] | [ ] |
| 7 | Missing Logout feature | [ ] | [ ] |
| 8 | Missing AuthMethod subtypes (EnvVar/Terminal polymorphism) | [ ] | [ ] |
| 9 | Missing base types (Position/Range/Cost/Usage etc.) | [ ] | [ ] |
| 10 | Missing Capability type extensions | [ ] | [ ] |
| 11 | Missing PromptRequest.message_id field | [ ] | [ ] |
| 12 | Missing NewSessionRequest/LoadSessionRequest.additional_directories field | [ ] | [ ] |
| 13 | ContentBlock missing annotations field | [ ] | [ ] |
| 14 | ToolCallContent polymorphic discriminator alignment (use type field) | [ ] | [ ] |
| 15 | RequestPermissionOutcome discriminator alignment (use outcome field) | [ ] | [ ] |
| 16 | RequestPermissionRequest.tool_call type from object to ToolCallUpdate | [ ] | [ ] |
