# API Reference

This document provides a reference for the main classes and methods in the PHP AI Client.

## Table of Contents

- [AiClient](#aiclient)
- [PromptBuilder](#promptbuilder)
- [MessageBuilder](#messagebuilder)
- [Results](#results)
- [Messages](#messages)
- [Files](#files)
- [Tools](#tools)
- [Providers](#providers)

## AiClient

The main entry point for all AI operations.

### Static Methods

#### `prompt()`

Creates a new prompt builder for fluent API usage.

```php
public static function prompt(
    string|Message|null $text = null
): PromptBuilder
```

**Parameters:**
- `$text` - Optional initial prompt text or message

**Returns:** `PromptBuilder` instance

**Example:**
```php
$builder = AiClient::prompt('Hello, AI!');
```

#### `message()`

Creates a new message builder.

```php
public static function message($input = null): MessageBuilder
```

**Parameters:**
- `$input` - Optional initial message input

**Returns:** `MessageBuilder` instance

#### `defaultRegistry()`

Gets the default provider registry instance.

```php
public static function defaultRegistry(): ProviderRegistry
```

**Returns:** The provider registry

#### `isConfigured()`

Checks if a provider is configured and ready to use.

```php
public static function isConfigured(
    ProviderAvailabilityInterface $availability
): bool
```

**Parameters:**
- `$availability` - Provider availability instance

**Returns:** True if configured, false otherwise

#### `setEventDispatcher()`

Sets the PSR-14 event dispatcher for lifecycle events.

```php
public static function setEventDispatcher(
    ?EventDispatcherInterface $eventDispatcher
): void
```

**Parameters:**
- `$eventDispatcher` - PSR-14 event dispatcher or null to disable

#### `generateTextResult()`

Generates text using the traditional API.

```php
public static function generateTextResult(
    string|MessagePart|MessagePart[]|Message|Message[] $prompt,
    ModelInterface|ModelConfig|null $modelOrConfig = null,
    ?ProviderRegistry $registry = null
): GenerativeAiResult
```

**Parameters:**
- `$prompt` - The prompt input
- `$modelOrConfig` - Optional model instance, model configuration, or null for auto-discovery
- `$registry` - Optional custom registry (defaults to the default registry)

**Returns:** `GenerativeAiResult` with text generation result

#### `streamGenerateTextResult()`

Streams text generation results as they're produced.

```php
public static function streamGenerateTextResult(
    string|MessagePart|MessagePart[]|Message|Message[] $prompt,
    ModelInterface|ModelConfig|null $modelOrConfig = null,
    ?ProviderRegistry $registry = null
): Generator<GenerativeAiResult>
```

**Parameters:**
- `$prompt` - The prompt input
- `$modelOrConfig` - Optional model instance, model configuration, or null for auto-discovery
- `$registry` - Optional custom registry (defaults to the default registry)

**Returns:** Generator yielding `GenerativeAiResult` chunks

#### `generateImageResult()`

Generates an image using the traditional API.

```php
public static function generateImageResult(
    string|MessagePart|MessagePart[]|Message|Message[] $prompt,
    ModelInterface|ModelConfig|null $modelOrConfig = null,
    ?ProviderRegistry $registry = null
): GenerativeAiResult
```

**Parameters:**
- `$prompt` - The prompt input
- `$modelOrConfig` - Optional model instance, model configuration, or null for auto-discovery
- `$registry` - Optional custom registry (defaults to the default registry)

**Returns:** `GenerativeAiResult` with image generation result

## PromptBuilder

Fluent API builder for constructing and executing AI prompts.

### Message Content Methods

#### `withText()`

Adds text to the prompt.

```php
public function withText(string $text): self
```

#### `withFile()`

Adds a file from various sources. Accepts:
- File object
- Local file path string
- URL string (remote file)
- Base64-encoded data string
- Data URI string (data:mime/type;base64,data)

```php
public function withFile($file, ?string $mimeType = null): self
```

**Parameters:**
- `$file` - File object or string (path, URL, base64, or data URI)
- `$mimeType` - Optional MIME type (required for base64 data, optional for others)

**Examples:**
```php
// From file path
->withFile('/path/to/image.jpg', 'image/jpeg')

// From URL
->withFile('https://example.com/image.jpg')

// From base64 data
->withFile($base64Data, 'image/png')

// From data URI
->withFile('data:image/png;base64,iVBORw0K...')
```

#### `withFunctionResponse()`

Adds a function response.

```php
public function withFunctionResponse(FunctionResponse $functionResponse): self
```

#### `withHistory()`

Adds message history for chat context.

```php
public function withHistory(Message ...$messages): self
```

### Model Selection Methods

#### `usingModel()`

Uses a specific model instance.

```php
public function usingModel(ModelInterface $model): self
```

#### `usingProvider()`

Uses any model from a specific provider.

```php
public function usingProvider(string $providerIdOrClassName): self
```

#### `usingModelConfig()`

Applies model configuration.

```php
public function usingModelConfig(ModelConfig $config): self
```

#### `usingModelPreference()`

Sets model preference order for automatic selection.

```php
public function usingModelPreference(
    string|array ...$preferences
): self
```

### Configuration Methods

#### `usingTemperature()`

Sets the temperature parameter (0.0-1.0).

```php
public function usingTemperature(float $temperature): self
```

#### `usingMaxTokens()`

Sets the maximum number of tokens to generate.

```php
public function usingMaxTokens(int $maxTokens): self
```

#### `usingTopP()`

Sets the top P (nucleus sampling) parameter.

```php
public function usingTopP(float $topP): self
```

#### `usingTopK()`

Sets the top K parameter.

```php
public function usingTopK(int $topK): self
```

#### `usingSystemInstruction()`

Sets system-level instructions.

```php
public function usingSystemInstruction(string $systemInstruction): self
```

#### `usingCandidateCount()`

Sets the number of candidates to generate.

```php
public function usingCandidateCount(int $candidateCount): self
```

#### `usingStopSequences()`

Sets stop sequences for generation.

```php
public function usingStopSequences(string ...$stopSequences): self
```

#### `usingFunctionDeclarations()`

Adds function declarations for function calling.

```php
public function usingFunctionDeclarations(
    FunctionDeclaration ...$functionDeclarations
): self
```

#### `usingPresencePenalty()`

Sets presence penalty to reduce repetition.

```php
public function usingPresencePenalty(float $presencePenalty): self
```

#### `usingFrequencyPenalty()`

Sets frequency penalty to reduce frequent tokens.

```php
public function usingFrequencyPenalty(float $frequencyPenalty): self
```

#### `usingWebSearch()`

Enables web search capabilities.

```php
public function usingWebSearch(WebSearch $webSearch): self
```

#### `usingTopLogprobs()`

Sets the number of top token log probabilities to return.

```php
public function usingTopLogprobs(?int $topLogprobs): self
```

### Output Format Methods

#### `asJsonResponse()`

Requests JSON output with optional schema.

```php
public function asJsonResponse(?array $schema = null): self
```

#### `asOutputMimeType()`

Sets the output MIME type.

```php
public function asOutputMimeType(string $mimeType): self
```

#### `asOutputSchema()`

Sets the output schema.

```php
public function asOutputSchema(array $schema): self
```

#### `asOutputModalities()`

Sets the output modalities.

```php
public function asOutputModalities(ModalityEnum ...$modalities): self
```

### Execution Methods

#### `generateText()`

Generates and returns text.

```php
public function generateText(): string
```

#### `generateTexts()`

Generates multiple text candidates.

```php
public function generateTexts(?int $candidateCount = null): array
```

#### `generateImage()`

Generates and returns an image file.

```php
public function generateImage(): File
```

#### `generateImages()`

Generates multiple image candidates.

```php
public function generateImages(?int $candidateCount = null): array
```

#### `generateTextResult()`

Generates and returns the full result object.

```php
public function generateTextResult(): GenerativeAiResult
```

#### `generateImageResult()`

Generates and returns the full image result object.

```php
public function generateImageResult(): GenerativeAiResult
```

### Capability Check Methods

#### `isSupported()`

Checks if a capability is supported.

```php
public function isSupported(?CapabilityEnum $capability = null): bool
```

#### `isSupportedForTextGeneration()`

Checks if text generation is supported.

```php
public function isSupportedForTextGeneration(): bool
```

#### `isSupportedForImageGeneration()`

Checks if image generation is supported.

```php
public function isSupportedForImageGeneration(): bool
```

## MessageBuilder

Builder for creating messages with multiple parts.

### Methods

#### `usingRole()`

Sets the message role.

```php
public function usingRole(MessageRoleEnum $role): self
```

#### `usingUserRole()`

Sets the role to USER.

```php
public function usingUserRole(): self
```

#### `usingModelRole()`

Sets the role to MODEL.

```php
public function usingModelRole(): self
```

#### `withText()`

Adds text to the message.

```php
public function withText(string $text): self
```

#### `withFile()`

Adds a file to the message.

```php
public function withFile($file, ?string $mimeType = null): self
```

#### `withFunctionCall()`

Adds a function call to the message.

```php
public function withFunctionCall(FunctionCall $functionCall): self
```

#### `withFunctionResponse()`

Adds a function response to the message.

```php
public function withFunctionResponse(FunctionResponse $functionResponse): self
```

#### `get()`

Builds and returns the message.

```php
public function get(): Message
```

## Results

### GenerativeAiResult

Represents the result of a generative AI operation.

#### Methods

```php
public function getId(): string
public function getCandidates(): array
public function getTokenUsage(): TokenUsage
public function getProviderMetadata(): ProviderMetadata
public function getModelMetadata(): ModelMetadata

// Transformation methods
public function toText(): string
public function toTexts(): array
public function toFile(): File
public function toFiles(): array
public function toMessage(): Message
public function toMessages(): array
```

### Candidate

Represents a single response candidate.

#### Methods

```php
public function getMessage(): Message
public function getFinishReason(): FinishReasonEnum
public function getTokenCount(): int
```

### TokenUsage

Represents token usage statistics.

#### Methods

```php
public function getPromptTokens(): int
public function getCompletionTokens(): int
public function getTotalTokens(): int
```

## Messages

### Message

Represents a message in a conversation.

#### Methods

```php
public function getRole(): MessageRoleEnum
public function getParts(): array // Returns MessagePart[]
```

#### Static Factory Methods

```php
Message::user(string $text): UserMessage
Message::model(string $text): ModelMessage
```

### MessagePart

Represents a part of a message.

#### Methods

```php
public function getType(): MessagePartTypeEnum
public function getText(): ?string
public function getFile(): ?File
public function getFunctionCall(): ?FunctionCall
public function getFunctionResponse(): ?FunctionResponse
```

#### Static Factory Methods

```php
MessagePart::text(string $text): MessagePart
MessagePart::file(File $file): MessagePart
MessagePart::functionCall(FunctionCall $call): MessagePart
MessagePart::functionResponse(FunctionResponse $response): MessagePart
```

## Files

### File

Represents a file (image, audio, video, etc.).

#### Methods

```php
public function getFileType(): FileTypeEnum
public function getMimeType(): string
public function getUrl(): ?string
public function getBase64Data(): ?string
```

#### Static Factory Methods

```php
File::fromPath(string $path, ?string $mimeType = null): File
File::fromUrl(string $url, ?string $mimeType = null): File
File::fromBase64(string $base64Data, string $mimeType): File
```

## Tools

### FunctionDeclaration

Declares a function that the model can call.

#### Constructor

```php
public function __construct(
    string $name,
    string $description,
    mixed $parameters
)
```

#### Methods

```php
public function getName(): string
public function getDescription(): string
public function getParameters(): mixed
```

### FunctionCall

Represents a function call from the model.

#### Methods

```php
public function getId(): ?string
public function getName(): ?string
public function getArgs(): array
```

### FunctionResponse

Represents the response to a function call.

#### Constructor

```php
public function __construct(
    ?string $id,
    ?string $name,
    mixed $response
)
```

#### Methods

```php
public function getId(): ?string
public function getName(): ?string
public function getResponse(): mixed
```

### WebSearch

Configures web search capabilities.

#### Constructor

```php
public function __construct(
    array $allowedDomains = [],
    array $disallowedDomains = []
)
```

#### Methods

```php
public function getAllowedDomains(): array
public function getDisallowedDomains(): array
```

## Providers

### ProviderRegistry

Manages available providers and models.

#### Methods

```php
public function registerProvider(string $className): void
public function getRegisteredProviderIds(): array
public function hasProvider(string $idOrClassName): bool
public function getProviderClassName(string $id): string
public function isProviderConfigured(string $idOrClassName): bool
public function getProviderModel(
    string $idOrClassName,
    string $modelId,
    ModelConfig|array $modelConfig = []
): ModelInterface
public function findProviderModelsMetadataForSupport(
    string $idOrClassName,
    ModelRequirements $modelRequirements
): array
public function findModelsMetadataForSupport(
    ModelRequirements $modelRequirements
): array
```

### Provider Classes

Built-in provider classes:

```php
WordPress\AiClient\ProviderImplementations\Google\GoogleProvider
WordPress\AiClient\ProviderImplementations\OpenAi\OpenAiProvider
WordPress\AiClient\ProviderImplementations\Anthropic\AnthropicProvider
```

#### Static Methods

```php
public static function metadata(): ProviderMetadata
public static function model(
    string $modelId,
    ModelConfig|array $modelConfig = []
): ModelInterface
public static function availability(): ProviderAvailabilityInterface
public static function modelMetadataDirectory(): ModelMetadataDirectoryInterface
```

### ModelConfig

Configuration for model behavior.

#### Methods

```php
public function setTemperature(float $temperature): void
public function getTemperature(): ?float
public function setMaxTokens(int $maxTokens): void
public function getMaxTokens(): ?int
public function setTopP(float $topP): void
public function getTopP(): ?float
public function setTopK(int $topK): void
public function getTopK(): ?int
public function setSystemInstruction(string $systemInstruction): void
public function getSystemInstruction(): ?string
public function setCandidateCount(int $candidateCount): void
public function getCandidateCount(): ?int
public function setOutputMimeType(string $outputMimeType): void
public function getOutputMimeType(): ?string
public function setOutputSchema(array $outputSchema): void
public function getOutputSchema(): ?array
public function setTools(array $tools): void
public function getTools(): array
```

#### Static Factory Method

```php
public static function fromArray(array $data): ModelConfig
```

### ModelMetadata

Metadata about a model's capabilities.

#### Methods

```php
public function getId(): string
public function getName(): string
public function getSupportedCapabilities(): array // CapabilityEnum[]
public function getSupportedOptions(): array // SupportedOption[]
```

## Enums

### CapabilityEnum

AI model capabilities:
- `TEXT_GENERATION`
- `IMAGE_GENERATION`
- `TEXT_TO_SPEECH_CONVERSION`
- `SPEECH_GENERATION`
- `MUSIC_GENERATION`
- `VIDEO_GENERATION`
- `EMBEDDING_GENERATION`
- `CHAT_HISTORY`

### MessageRoleEnum

Message roles:
- `USER` - User message
- `MODEL` - Model response

### MessagePartTypeEnum

Message part types:
- `TEXT`
- `FILE`
- `FUNCTION_CALL`
- `FUNCTION_RESPONSE`

### ModalityEnum

Input/output modalities:
- `TEXT`
- `DOCUMENT`
- `IMAGE`
- `AUDIO`
- `VIDEO`

### FileTypeEnum

File types:
- `INLINE` - Base64-encoded inline data
- `REMOTE` - URL to remote file

### FinishReasonEnum

Reasons why generation finished:
- `STOP` - Natural stop
- `LENGTH` - Max tokens reached
- `CONTENT_FILTER` - Content filtered
- `TOOL_CALLS` - Stopped for tool calls
- `ERROR` - Error occurred

## Events

### BeforeGenerateResultEvent

Dispatched before generating a result.

#### Methods

```php
public function getModel(): ModelInterface
public function getMessages(): array // Message[]
public function getCapability(): CapabilityEnum
```

### AfterGenerateResultEvent

Dispatched after generating a result.

#### Methods

```php
public function getResult(): GenerativeAiResult
```

## Exceptions

### Exception Hierarchy

```
Exception
└── AiClientExceptionInterface
    ├── InvalidArgumentException
    ├── RuntimeException
    └── ResponseException (in Providers\Http\Exception namespace)
```

All custom exceptions implement `AiClientExceptionInterface` for unified exception handling.

## Next Steps

- See [Usage Guide](./USAGE.md) for practical examples
- Read [Advanced Features](./ADVANCED.md) for extended functionality
- Review [Architecture](./ARCHITECTURE.md) for system design
