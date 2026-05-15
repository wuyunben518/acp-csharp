# AGENTS.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Unofficial C# SDK for ACP (Agent Client Protocol), a standardized protocol for editor/IDE-to-coding-agent communication. The library is published as the `AgentClientProtocol` NuGet package.

## Build & Run

```bash
dotnet build                                    # Build the library
dotnet build src/AgentClientProtocol             # Build only the library
dotnet pack src/AgentClientProtocol              # Create NuGet package
dotnet run --project sandbox/ConsoleApp1         # Run the example client
```

Target frameworks: net10.0, net9.0, net8.0, netstandard2.1. Uses .NET 10 SDK.

## Architecture

**Communication model**: JSON-RPC over `TextReader`/`TextWriter` (stdin/stdout), with `JsonRpcEndpoint` as the transport layer.

**Two connection roles**:
- `ClientSideConnection` — wraps an `IAcpClient` implementation. Implements `IAcpAgent` to forward agent-bound requests. Created with a factory `Func<IAcpAgent, IAcpClient>` so the client can hold a reference to the connection.
- `AgentSideConnection` — wraps an `IAcpAgent` implementation. Handles client-bound requests.

**Method constants**: `AgentMethods` and `ClientMethods` in `Constants.cs` define the JSON-RPC method names.

**Schema types** (`Schema/` folder): Request/response records with `[JsonPropertyName]` attributes. Polymorphic types use custom `JsonConverter` implementations that dispatch on a discriminator field (e.g., `ContentBlock` dispatches on `type`, `SessionUpdate` dispatches on `sessionUpdate`).

**AOT compatibility**: Uses `AcpJsonSerializerContext` (a `JsonSerializerContext` with `[JsonSerializable]` attributes) for all serialization. This enables Native AOT compilation support. Do not use reflection-based `JsonSerializer.Serialize<T>` / `Deserialize<T>` — always use `AcpJsonSerializerContext.Default.Options.GetTypeInfo<T>()`.

**Extensibility**: Both `IAcpClient` and `IAcpAgent` have `ExtMethodAsync` and `ExtNotificationAsync` methods for custom JSON-RPC methods not part of the core protocol.
