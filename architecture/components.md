# Semantic Kernel Python: Component Architecture

## Introduction

This document describes the key components of the Semantic Kernel Python implementation, their responsibilities, relationships, and interactions. Understanding these components is essential for effectively working with and extending the framework.

## Core Components

Semantic Kernel Python is organized into several core components, each with specific responsibilities and interfaces.

### Kernel

**Purpose**: Central orchestration point that manages plugins, services, and function execution.

**Key Responsibilities**:
- Register and manage plugins and functions
- Manage AI service clients
- Provide function invocation pipeline
- Coordinate interaction between components
- Manage memory services

**Key Interfaces**:
- `invoke()`: Execute functions
- `invoke_stream()`: Execute functions with streaming responses
- `invoke_prompt()`: Create and execute functions from prompts
- `add_plugin()`: Register plugins with the kernel

**Component Diagram**:

```mermaid
classDiagram
    class Kernel {
        +plugins: dict
        +services: dict
        +ai_service_selector: AIServiceSelector
        +function_invocation_filters: list
        +prompt_rendering_filters: list
        +auto_function_invocation_filters: list
        +invoke()
        +invoke_stream()
        +invoke_prompt()
        +invoke_prompt_stream()
        +add_plugin()
        +add_service()
    }
    
    Kernel --|> KernelFilterExtension
    Kernel --|> KernelFunctionExtension
    Kernel --|> KernelServicesExtension
    Kernel --|> KernelReliabilityExtension
    
    class KernelFilterExtension {
        +function_invocation_filters: list
        +prompt_rendering_filters: list
        +auto_function_invocation_filters: list
        +add_filter()
    }
    
    class KernelFunctionExtension {
        +plugins: dict
        +add_plugin()
        +get_function()
    }
    
    class KernelServicesExtension {
        +services: dict
        +ai_service_selector: AIServiceSelector
        +add_service()
        +get_service()
    }
    
    class KernelReliabilityExtension {
        +retry_mechanism: RetryMechanism
        +configure_retry()
    }
```

### Functions

**Purpose**: Encapsulate executable units of work, whether code-based or AI-based.

**Types**:
- **KernelFunction**: Base class for all functions
- **KernelFunctionFromMethod**: Wraps Python methods as functions
- **KernelFunctionFromPrompt**: Creates functions from prompt templates

**Key Responsibilities**:
- Execute business logic
- Interact with AI services
- Process inputs and produce outputs
- Define parameter requirements

**Key Interfaces**:
- `invoke()`: Execute the function
- `invoke_stream()`: Execute with streaming results

**Component Diagram**:

```mermaid
classDiagram
    class KernelFunction {
        +plugin_name: str
        +name: str
        +description: str
        +parameters: list
        +metadata: dict
        +invoke()
        +invoke_stream()
    }
    
    class KernelFunctionFromMethod {
        +method: callable
        +invoke()
        +invoke_stream()
    }
    
    class KernelFunctionFromPrompt {
        +prompt: str
        +template_format: str
        +prompt_template: PromptTemplateBase
        +execution_settings: dict
        +invoke()
        +invoke_stream()
    }
    
    KernelFunction <|-- KernelFunctionFromMethod
    KernelFunction <|-- KernelFunctionFromPrompt
```

### Plugins

**Purpose**: Organize related functions into cohesive units that can be registered with the kernel.

**Key Responsibilities**:
- Contain related functions
- Provide domain-specific functionality
- Enable discovery and registration of functions

**Key Interfaces**:
- KernelPlugin: Container for functions

**Component Diagram**:

```mermaid
classDiagram
    class KernelPlugin {
        +name: str
        +description: str
        +functions: dict
        +add_function()
        +get_function()
    }
    
    KernelPlugin *-- KernelFunction
```

### Memory

**Purpose**: Provide storage and retrieval of information with semantic search capabilities.

**Key Components**:
- Memory stores (semantic memory)
- Collection management
- Embedding services

**Key Interfaces**:
- `save_information()`: Store information
- `search()`: Search for relevant information
- `save_reference()`: Store references to information

**Component Diagram**:

```mermaid
classDiagram
    class SemanticTextMemory {
        +save_information()
        +save_reference()
        +search()
    }
    
    class MemoryStore {
        +create_collection()
        +get_collection()
        +upsert()
        +search()
    }
    
    class EmbeddingGeneratorBase {
        +generate_embeddings()
    }
    
    SemanticTextMemory --> MemoryStore
    SemanticTextMemory --> EmbeddingGeneratorBase
```

### Connectors

**Purpose**: Interface with external services and systems.

**Types**:
- **AI Connectors**: Connect to AI services (OpenAI, Azure, etc.)
- **Memory Connectors**: Connect to vector databases
- **Search Connectors**: Connect to search engines

**Key Responsibilities**:
- Abstract external service details
- Handle authentication and communication
- Convert between SK formats and external formats

**Component Diagram**:

```mermaid
classDiagram
    class AIServiceClientBase {
        +get_text_completion()
        +get_chat_completion()
    }
    
    class EmbeddingGeneratorBase {
        +generate_embeddings()
    }
    
    class TextCompletionClientBase {
        +complete()
        +complete_stream()
    }
    
    class ChatCompletionClientBase {
        +complete_chat()
        +complete_chat_stream()
    }
    
    AIServiceClientBase <|-- OpenAITextCompletion
    AIServiceClientBase <|-- AzureOpenAIChatCompletion
    AIServiceClientBase <|-- AnthropicChatCompletion
    
    EmbeddingGeneratorBase <|-- OpenAITextEmbedding
    EmbeddingGeneratorBase <|-- AzureOpenAIEmbedding
    
    TextCompletionClientBase <|-- OpenAITextCompletion
    
    ChatCompletionClientBase <|-- OpenAIChatCompletion
    ChatCompletionClientBase <|-- AzureOpenAIChatCompletion
    ChatCompletionClientBase <|-- AnthropicChatCompletion
```

### Template Engine

**Purpose**: Process prompt templates with variable substitution.

**Key Responsibilities**:
- Parse prompt templates
- Resolve variables in templates
- Support different template formats

**Key Components**:
- PromptTemplateEngine
- Template blocks and variables

**Component Diagram**:

```mermaid
classDiagram
    class PromptTemplateEngine {
        +render()
        +render_async()
    }
    
    class PromptTemplateBase {
        +template: str
        +render()
        +render_async()
    }
    
    class TemplateTokenizer {
        +tokenize()
    }
    
    class Block {
        +render()
    }
    
    PromptTemplateEngine --> TemplateTokenizer
    TemplateTokenizer --> Block
    PromptTemplateBase --> PromptTemplateEngine
```

### Planners

**Purpose**: Orchestrate function execution to achieve complex goals.

**Types**:
- **Sequential Planner**: Creates a sequence of steps
- **Stepwise Planner**: Dynamically determines next steps

**Key Responsibilities**:
- Create execution plans
- Determine functions to call
- Handle execution flow

**Component Diagram**:

```mermaid
classDiagram
    class PlannerBase {
        +create_plan()
        +execute_plan()
    }
    
    class Plan {
        +steps: list
        +state: dict
        +execute()
    }
    
    class SequentialPlanner {
        +create_plan()
    }
    
    class StepwisePlanner {
        +execute_plan()
    }
    
    PlannerBase <|-- SequentialPlanner
    PlannerBase <|-- StepwisePlanner
    SequentialPlanner --> Plan
    StepwisePlanner --> Plan
```

### Content System

**Purpose**: Manage and manipulate content structures for interaction with AI services.

**Key Components**:
- ChatMessage
- ChatHistory
- StreamingContent

**Key Responsibilities**:
- Represent chat messages and conversations
- Handle streaming content
- Support function calls and results

**Component Diagram**:

```mermaid
classDiagram
    class ChatHistory {
        +messages: list
        +add_message()
        +add_user_message()
        +add_assistant_message()
    }
    
    class ChatMessageContent {
        +role: str
        +content: str
        +items: list
    }
    
    class StreamingChatMessageContent {
        +role: str
        +content: str
        +choice_index: int
        +append()
    }
    
    class FunctionCallContent {
        +name: str
        +arguments: dict
        +to_kernel_arguments()
    }
    
    class FunctionResultContent {
        +name: str
        +result: Any
    }
    
    ChatHistory *-- ChatMessageContent
    ChatMessageContent <|-- StreamingChatMessageContent
    ChatMessageContent *-- FunctionCallContent
    ChatMessageContent *-- FunctionResultContent
```

## Component Relationships

The following diagram illustrates the key relationships between Semantic Kernel Python components:

```mermaid
graph TD
    Kernel --> Functions[Functions]
    Kernel --> Plugins[Plugins]
    Kernel --> Memory[Memory]
    Kernel --> Planners[Planners]
    Kernel --> Services[Services]
    
    Plugins --> Functions
    
    Functions --> TemplateEngine[Template Engine]
    Functions --> AIServices[AI Services]
    
    Memory --> VectorStores[Vector Stores]
    Memory --> EmbeddingServices[Embedding Services]
    
    Planners --> Functions
    
    AIServices --> ContentSystem[Content System]
    
    TemplateEngine --> ContentSystem
```

## Extension Points

Semantic Kernel Python is designed to be extensible at multiple levels:

1. **Custom Functions**: Create native functions by implementing the `KernelFunction` interface
2. **Custom Plugins**: Create plugins by grouping related functions
3. **Custom AI Connectors**: Implement `AIServiceClientBase` to connect to new AI services
4. **Custom Memory Connectors**: Implement `MemoryStore` for new vector databases
5. **Custom Planners**: Extend `PlannerBase` to create new planning strategies
6. **Custom Filters**: Implement function invocation filters to intercept and modify function execution

## Conclusion

The component architecture of Semantic Kernel Python provides a flexible and extensible foundation for building AI-powered applications. By understanding these components and their relationships, developers can effectively use and extend the framework to meet their specific needs.

The modular design allows for replacing or extending individual components without affecting the entire system, making it adaptable to a wide range of use cases and integration scenarios.