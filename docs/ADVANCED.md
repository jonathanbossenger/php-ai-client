# Advanced Features

This guide covers advanced features of the PHP AI Client, including custom providers, operations, streaming, and extending the SDK.

## Table of Contents

- [Custom Providers](#custom-providers)
- [Long-Running Operations](#long-running-operations)
- [Streaming Responses](#streaming-responses)
- [Model Discovery](#model-discovery)
- [Custom HTTP Transport](#custom-http-transport)
- [Function Calling](#function-calling)
- [Provider Metadata](#provider-metadata)
- [Extending the SDK](#extending-the-sdk)

## Custom Providers

Create custom providers to integrate with new AI services or local models.

### Implementing a Provider

```php
namespace YourNamespace;

use WordPress\AiClient\Providers\Contracts\ProviderInterface;
use WordPress\AiClient\Providers\Contracts\ProviderAvailabilityInterface;
use WordPress\AiClient\Providers\Contracts\ModelMetadataDirectoryInterface;
use WordPress\AiClient\Providers\DTO\ProviderMetadata;
use WordPress\AiClient\Providers\Models\Contracts\ModelInterface;
use WordPress\AiClient\Providers\Models\DTO\ModelConfig;
use WordPress\AiClient\Providers\Enums\ProviderTypeEnum;

class CustomProvider implements ProviderInterface
{
    /**
     * Returns provider metadata.
     */
    public static function metadata(): ProviderMetadata
    {
        return new ProviderMetadata(
            'custom',                        // Provider ID
            'Custom AI Provider',            // Display name
            ProviderTypeEnum::CLOUD         // Provider type
        );
    }

    /**
     * Creates a model instance.
     */
    public static function model(
        string $modelId,
        ModelConfig|array $modelConfig = []
    ): ModelInterface {
        // Return your model implementation
        return new CustomModel($modelId, $modelConfig);
    }

    /**
     * Returns availability checker.
     */
    public static function availability(): ProviderAvailabilityInterface
    {
        return new CustomProviderAvailability();
    }

    /**
     * Returns model metadata directory.
     */
    public static function modelMetadataDirectory(): ModelMetadataDirectoryInterface
    {
        return new CustomModelMetadataDirectory();
    }
}
```

### Implementing Provider Availability

```php
use WordPress\AiClient\Providers\Contracts\ProviderAvailabilityInterface;

class CustomProviderAvailability implements ProviderAvailabilityInterface
{
    public function isConfigured(): bool
    {
        // Check if API key or credentials are available
        return !empty(getenv('CUSTOM_API_KEY'));
    }
}
```

### Implementing Model Metadata Directory

```php
use WordPress\AiClient\Providers\Contracts\ModelMetadataDirectoryInterface;
use WordPress\AiClient\Providers\Models\DTO\ModelMetadata;
use WordPress\AiClient\Providers\Models\DTO\SupportedOption;
use WordPress\AiClient\Providers\Models\Enums\CapabilityEnum;

class CustomModelMetadataDirectory implements ModelMetadataDirectoryInterface
{
    public function listModelMetadata(): array
    {
        return [
            new ModelMetadata(
                'custom-model-v1',
                'Custom Model v1',
                [CapabilityEnum::TEXT_GENERATION],
                [
                    new SupportedOption('temperature', [0.0, 1.0]),
                    new SupportedOption('maxTokens', [1, 4096]),
                ]
            ),
        ];
    }

    public function hasModelMetadata(string $modelId): bool
    {
        foreach ($this->listModelMetadata() as $metadata) {
            if ($metadata->getId() === $modelId) {
                return true;
            }
        }
        return false;
    }

    public function getModelMetadata(string $modelId): ModelMetadata
    {
        foreach ($this->listModelMetadata() as $metadata) {
            if ($metadata->getId() === $modelId) {
                return $metadata;
            }
        }
        throw new \RuntimeException("Model not found: {$modelId}");
    }
}
```

### Implementing a Model

```php
use WordPress\AiClient\Providers\Models\Contracts\ModelInterface;
use WordPress\AiClient\Providers\Models\Contracts\TextGenerationModelInterface;
use WordPress\AiClient\Providers\Models\DTO\ModelConfig;
use WordPress\AiClient\Providers\Models\DTO\ModelMetadata;
use WordPress\AiClient\Results\DTO\GenerativeAiResult;
use WordPress\AiClient\Messages\DTO\Message;

class CustomModel implements ModelInterface, TextGenerationModelInterface
{
    private string $modelId;
    private ModelConfig $config;

    public function __construct(string $modelId, ModelConfig|array $modelConfig = [])
    {
        $this->modelId = $modelId;
        $this->config = is_array($modelConfig) 
            ? ModelConfig::fromArray($modelConfig) 
            : $modelConfig;
    }

    public function metadata(): ModelMetadata
    {
        // Return metadata for this model
        return CustomProvider::modelMetadataDirectory()
            ->getModelMetadata($this->modelId);
    }

    public function setConfig(ModelConfig $config): void
    {
        $this->config = $config;
    }

    public function getConfig(): ModelConfig
    {
        return $this->config;
    }

    public function generateTextResult(array $prompt): GenerativeAiResult
    {
        // Implement your model's text generation logic
        // Use HTTP transporter to make API calls
        // Return GenerativeAiResult
    }

    public function streamGenerateTextResult(array $prompt): \Generator
    {
        // Implement streaming if supported
        // Yield GenerativeAiResult objects as they arrive
    }
}
```

### Registering Your Provider

```php
use WordPress\AiClient\AiClient;

$registry = AiClient::defaultRegistry();
$registry->registerProvider(CustomProvider::class);

// Now you can use it
$text = AiClient::prompt('Hello!')
    ->usingProvider('custom')
    ->generateText();
```

## Long-Running Operations

Some operations may take several minutes. The SDK supports tracking operation status.

### Generating an Operation

```php
use WordPress\AiClient\AiClient;

// Start a long-running operation
$operation = AiClient::prompt('Generate a complex video.')
    ->generateOperation();

// Check operation state
$state = $operation->getState();
echo "Operation state: " . $state->value . PHP_EOL;

// Get operation ID for later retrieval
$operationId = $operation->getId();
```

### Operation States

Operations can be in the following states:
- `STARTING` - Operation is initializing
- `PROCESSING` - Operation is in progress
- `SUCCEEDED` - Operation completed successfully
- `FAILED` - Operation failed
- `CANCELED` - Operation was canceled

### Polling for Completion

```php
use WordPress\AiClient\Operations\Enums\OperationStateEnum;

$operationId = 'operation-123';

while (true) {
    $operation = $provider->operationsHandler()->getOperation($operationId);
    
    $state = $operation->getState();
    
    if ($state === OperationStateEnum::SUCCEEDED) {
        $result = $operation->getResult();
        echo "Operation completed: " . $result->toText();
        break;
    } elseif ($state === OperationStateEnum::FAILED) {
        echo "Operation failed!";
        break;
    }
    
    echo "Still processing... (" . $state->value . ")" . PHP_EOL;
    sleep(5); // Wait 5 seconds before checking again
}
```

## Streaming Responses

Stream text generation results as they're produced (if supported by the provider).

### Basic Streaming

```php
use WordPress\AiClient\AiClient;
use WordPress\AiClient\ProviderImplementations\OpenAi\OpenAiProvider;

$model = OpenAiProvider::model('gpt-4o');

$stream = AiClient::streamGenerateTextResult(
    'Write a long essay about artificial intelligence.',
    $model
);

foreach ($stream as $chunk) {
    // Each chunk is a GenerativeAiResult
    $text = $chunk->toText();
    echo $text; // Print text as it arrives
    flush();
}
```

### Streaming with the Fluent API

```php
$promptBuilder = AiClient::prompt('Tell me a long story.')
    ->usingModel(OpenAiProvider::model('gpt-4o'));

// Check if streaming is supported
if ($promptBuilder->isSupported()) {
    // Use the traditional API for streaming
    $model = OpenAiProvider::model('gpt-4o');
    $messages = [/* your messages */];
    
    $stream = AiClient::streamGenerateTextResult($messages, $model);
    
    foreach ($stream as $chunk) {
        echo $chunk->toText();
        flush();
    }
} else {
    // Fall back to non-streaming
    $text = $promptBuilder->generateText();
    echo $text;
}
```

## Model Discovery

Find models that match specific requirements.

### Finding Models by Capability

```php
use WordPress\AiClient\AiClient;
use WordPress\AiClient\Providers\Models\DTO\ModelRequirements;
use WordPress\AiClient\Providers\Models\Enums\CapabilityEnum;

$requirements = new ModelRequirements([
    CapabilityEnum::TEXT_GENERATION,
    CapabilityEnum::CHAT_HISTORY,
]);

$providerModelsMetadata = AiClient::defaultRegistry()
    ->findModelsMetadataForSupport($requirements);

foreach ($providerModelsMetadata as $providerModel) {
    $provider = $providerModel->getProvider();
    $models = $providerModel->getModels();
    
    echo "Provider: " . $provider->getName() . PHP_EOL;
    foreach ($models as $model) {
        echo "  - " . $model->getName() . " (" . $model->getId() . ")" . PHP_EOL;
    }
}
```

### Finding Models with Specific Options

```php
use WordPress\AiClient\Providers\Models\DTO\RequiredOption;

$requirements = new ModelRequirements(
    [CapabilityEnum::TEXT_GENERATION],
    [
        new RequiredOption('outputMimeType', 'application/json'),
        new RequiredOption('outputSchema', true),
    ]
);

$modelsMetadata = AiClient::defaultRegistry()
    ->findModelsMetadataForSupport($requirements);
```

### Finding Models for a Specific Provider

```php
$googleModels = AiClient::defaultRegistry()
    ->findProviderModelsMetadataForSupport(
        'google',
        new ModelRequirements([CapabilityEnum::IMAGE_GENERATION])
    );
```

## Custom HTTP Transport

Customize HTTP communication for specific needs.

### Custom HTTP Transporter

```php
use WordPress\AiClient\Providers\Http\Contracts\HttpTransporterInterface;
use WordPress\AiClient\Providers\Http\DTO\Request;
use WordPress\AiClient\Providers\Http\DTO\Response;
use WordPress\AiClient\Providers\Http\DTO\RequestOptions;

class CustomHttpTransporter implements HttpTransporterInterface
{
    public function send(Request $request, ?RequestOptions $options = null): Response
    {
        // Implement custom HTTP logic
        // Add custom headers, logging, rate limiting, etc.
        
        // Use PSR-18 client internally
        $psrRequest = $this->convertToPsr($request);
        $psrResponse = $this->client->sendRequest($psrRequest);
        
        return $this->convertFromPsr($psrResponse);
    }
}
```

### Using Custom Transporter with Models

```php
use WordPress\AiClient\Providers\Http\Contracts\WithHttpTransporterInterface;

$model = OpenAiProvider::model('gpt-4o');

if ($model instanceof WithHttpTransporterInterface) {
    $customTransporter = new CustomHttpTransporter();
    $model->setHttpTransporter($customTransporter);
}

$text = AiClient::prompt('Hello!')
    ->usingModel($model)
    ->generateText();
```

## Function Calling

Enable models to call predefined functions.

### Declaring Functions

```php
use WordPress\AiClient\Tools\DTO\FunctionDeclaration;

$getWeather = new FunctionDeclaration(
    'get_current_weather',
    'Get the current weather in a given location',
    [
        'type' => 'object',
        'properties' => [
            'location' => [
                'type' => 'string',
                'description' => 'The city and state, e.g. San Francisco, CA',
            ],
            'unit' => [
                'type' => 'string',
                'enum' => ['celsius', 'fahrenheit'],
                'description' => 'The temperature unit',
            ],
        ],
        'required' => ['location'],
    ]
);

$result = AiClient::prompt('What is the weather in Tokyo?')
    ->usingFunctionDeclarations($getWeather)
    ->generateTextResult();
```

### Handling Function Calls

```php
use WordPress\AiClient\Tools\DTO\FunctionResponse;

$candidates = $result->getCandidates();
$message = $candidates[0]->getMessage();

foreach ($message->getParts() as $part) {
    if ($part->getFunctionCall()) {
        $functionCall = $part->getFunctionCall();
        
        // Execute the function
        $functionName = $functionCall->getName();
        $args = $functionCall->getArgs();
        
        if ($functionName === 'get_current_weather') {
            $weatherData = $this->getCurrentWeather($args['location'], $args['unit'] ?? 'celsius');
            
            // Send response back to model
            $response = new FunctionResponse(
                $functionCall->getId(),
                $functionName,
                $weatherData
            );
            
            $finalResult = AiClient::prompt()
                ->withHistory($message)
                ->withFunctionResponse($response)
                ->generateTextResult();
        }
    }
}
```

## Provider Metadata

Access information about providers and models.

### Getting Provider Information

```php
use WordPress\AiClient\ProviderImplementations\Google\GoogleProvider;

$metadata = GoogleProvider::metadata();

echo "Provider ID: " . $metadata->getId() . PHP_EOL;
echo "Provider Name: " . $metadata->getName() . PHP_EOL;
echo "Provider Type: " . $metadata->getType()->value . PHP_EOL;
```

### Getting Model Information

```php
$model = GoogleProvider::model('gemini-2.5-flash');
$metadata = $model->metadata();

echo "Model ID: " . $metadata->getId() . PHP_EOL;
echo "Model Name: " . $metadata->getName() . PHP_EOL;

echo "Capabilities:" . PHP_EOL;
foreach ($metadata->getSupportedCapabilities() as $capability) {
    echo "  - " . $capability->value . PHP_EOL;
}

echo "Supported Options:" . PHP_EOL;
foreach ($metadata->getSupportedOptions() as $option) {
    echo "  - " . $option->getName() . PHP_EOL;
}
```

## Extending the SDK

### Custom Message Parts

Create custom message part types for specialized use cases:

```php
use WordPress\AiClient\Messages\DTO\MessagePart;
use WordPress\AiClient\Messages\Enums\MessagePartTypeEnum;

// Use existing message parts or extend for custom types
$textPart = MessagePart::text('Hello, world!');
$filePart = MessagePart::file($file);
```

### Custom Result Transformations

Transform results into custom formats:

```php
$result = AiClient::prompt('Explain AI.')
    ->generateTextResult();

// Custom transformation
class ResultTransformer
{
    public static function toMarkdown(GenerativeAiResult $result): string
    {
        $text = $result->toText();
        $usage = $result->getTokenUsage();
        
        return "## Response\n\n{$text}\n\n" .
               "---\n*Tokens used: {$usage->getTotalTokens()}*";
    }
}

$markdown = ResultTransformer::toMarkdown($result);
```

### Middleware Pattern

Implement middleware for request/response processing:

```php
use WordPress\AiClient\Events\BeforeGenerateResultEvent;
use WordPress\AiClient\Events\AfterGenerateResultEvent;

class RateLimitMiddleware
{
    private int $requestCount = 0;
    private int $maxRequests = 100;
    
    public function onBeforeGenerate(BeforeGenerateResultEvent $event): void
    {
        if ($this->requestCount >= $this->maxRequests) {
            throw new \RuntimeException('Rate limit exceeded');
        }
        $this->requestCount++;
    }
}

class CacheMiddleware
{
    private array $cache = [];
    
    public function onAfterGenerate(AfterGenerateResultEvent $event): void
    {
        $result = $event->getResult();
        $cacheKey = $this->generateCacheKey($event);
        $this->cache[$cacheKey] = $result;
    }
}
```

## Next Steps

- See the [API Reference](./API_REFERENCE.md) for detailed class documentation
- Review [Usage Examples](./USAGE.md) for common patterns
- Read the [Architecture](./ARCHITECTURE.md) document for system design details
