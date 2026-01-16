# Configuration

This guide covers configuration options for the PHP AI Client, including provider setup, authentication, and advanced configuration.

## Table of Contents

- [Provider Configuration](#provider-configuration)
- [Authentication Methods](#authentication-methods)
- [Model Configuration](#model-configuration)
- [HTTP Client Configuration](#http-client-configuration)
- [Event Dispatcher](#event-dispatcher)
- [Environment Variables](#environment-variables)

## Provider Configuration

The client automatically registers default providers (Google, OpenAI, Anthropic) but you can also register custom providers.

### Default Providers

These providers are available by default:

- **Google** (`google`) - Gemini models
- **OpenAI** (`openai`) - GPT models
- **Anthropic** (`anthropic`) - Claude models

### Checking Available Providers

```php
use WordPress\AiClient\AiClient;

$registry = AiClient::defaultRegistry();
$providerIds = $registry->getRegisteredProviderIds();

foreach ($providerIds as $providerId) {
    echo "Available provider: {$providerId}" . PHP_EOL;
}
```

### Checking Provider Configuration

```php
use WordPress\AiClient\ProviderImplementations\Google\GoogleProvider;

$isConfigured = AiClient::isConfigured(GoogleProvider::availability());

if ($isConfigured) {
    echo "Google provider is ready to use!";
} else {
    echo "Google provider is not configured. Set GOOGLE_API_KEY.";
}
```

### Registering Custom Providers

```php
use WordPress\AiClient\AiClient;
use YourNamespace\YourCustomProvider;

$registry = AiClient::defaultRegistry();
$registry->registerProvider(YourCustomProvider::class);
```

## Authentication Methods

Different providers support different authentication methods.

### API Key Authentication

Most cloud providers use API keys:

```php
// Set via environment variable (recommended)
putenv('GOOGLE_API_KEY=your-key-here');
putenv('OPENAI_API_KEY=your-key-here');
putenv('ANTHROPIC_API_KEY=your-key-here');
```

Or set in your system environment:

```bash
export GOOGLE_API_KEY="your-key-here"
export OPENAI_API_KEY="your-key-here"
export ANTHROPIC_API_KEY="your-key-here"
```

### Custom Authentication

For custom providers, you can implement your own authentication:

```php
use WordPress\AiClient\Providers\Http\Contracts\RequestAuthenticationInterface;
use WordPress\AiClient\Providers\Http\DTO\Request;

class CustomAuth implements RequestAuthenticationInterface
{
    public function authenticateRequest(Request $request): Request
    {
        // Add your authentication headers or modify the request
        return $request->withHeader('Authorization', 'Bearer your-token');
    }
    
    public static function getJsonSchema(): array
    {
        return [
            'type' => 'object',
            'properties' => [],
        ];
    }
}
```

## Model Configuration

Configure models using the `ModelConfig` class or fluent methods.

### Using ModelConfig Object

```php
use WordPress\AiClient\AiClient;
use WordPress\AiClient\Providers\Models\DTO\ModelConfig;

$config = new ModelConfig();
$config->setTemperature(0.7);
$config->setMaxTokens(500);
$config->setTopP(0.9);
$config->setSystemInstruction('You are a helpful assistant.');

$text = AiClient::prompt('Explain recursion.')
    ->usingModelConfig($config)
    ->generateText();
```

### Using Fluent Methods

```php
$text = AiClient::prompt('Write a story.')
    ->usingTemperature(0.8)
    ->usingMaxTokens(1000)
    ->usingTopP(0.95)
    ->usingTopK(40)
    ->usingSystemInstruction('You are a creative writer.')
    ->generateText();
```

### Available Configuration Options

| Option | Method | Type | Description |
|--------|--------|------|-------------|
| Temperature | `usingTemperature()` | float | Controls randomness (0.0-1.0) |
| Max Tokens | `usingMaxTokens()` | int | Maximum response length |
| Top P | `usingTopP()` | float | Nucleus sampling threshold |
| Top K | `usingTopK()` | int | Limits token selection pool |
| System Instruction | `usingSystemInstruction()` | string | Sets system-level context |
| Candidate Count | `usingCandidateCount()` | int | Number of responses to generate |
| Stop Sequences | `usingStopSequences()` | string[] | Stop generation at sequences |
| Presence Penalty | `usingPresencePenalty()` | float | Reduces repetition |
| Frequency Penalty | `usingFrequencyPenalty()` | float | Reduces frequent tokens |
| Top Logprobs | `usingTopLogprobs()` | int\|null | Number of top token probabilities |

### JSON Output

Request JSON responses with optional schema validation:

```php
$jsonString = AiClient::prompt('List 3 programming languages as JSON.')
    ->asJsonResponse([
        'type' => 'array',
        'items' => [
            'type' => 'object',
            'properties' => [
                'name' => ['type' => 'string'],
                'year' => ['type' => 'integer'],
            ],
            'required' => ['name', 'year'],
        ],
    ])
    ->generateText();

$data = json_decode($jsonString, true);
```

### Custom Output MIME Type

```php
$text = AiClient::prompt('Generate structured data.')
    ->asOutputMimeType('application/json')
    ->generateText();
```

### Function Declarations (Tools)

Declare functions that the model can call:

```php
use WordPress\AiClient\Tools\DTO\FunctionDeclaration;

$getCurrentWeather = new FunctionDeclaration(
    'getCurrentWeather',
    'Gets the current weather for a location',
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
            ],
        ],
        'required' => ['location'],
    ]
);

$text = AiClient::prompt('What is the weather in Paris?')
    ->usingFunctionDeclarations($getCurrentWeather)
    ->generateText();
```

### Web Search

Enable web search capabilities (if supported by the provider):

```php
use WordPress\AiClient\Tools\DTO\WebSearch;

$webSearch = new WebSearch(
    ['trusted-domain.com'], // Allowed domains
    ['blocked-domain.com']  // Disallowed domains
);

$text = AiClient::prompt('What are the latest AI developments?')
    ->usingWebSearch($webSearch)
    ->generateText();
```

## HTTP Client Configuration

### Request Options

Configure per-request HTTP options:

```php
use WordPress\AiClient\Providers\Http\DTO\RequestOptions;

$options = new RequestOptions();
$options->setTimeout(30.0);           // 30 second timeout
$options->setConnectTimeout(10.0);    // 10 second connect timeout
$options->setMaxRedirects(5);         // Allow up to 5 redirects

// Options are applied automatically by the HTTP transporter
```

### Custom HTTP Client

If you need to use a specific HTTP client:

```php
use WordPress\AiClient\Providers\Http\HttpTransporterFactory;
use YourNamespace\YourHttpClient;

// Create a custom transporter with your client
$client = new YourHttpClient();
$transporter = HttpTransporterFactory::create($client);

// Use with models that support WithHttpTransporterInterface
```

## Event Dispatcher

The client supports PSR-14 event dispatching for lifecycle hooks.

### Setting Up the Event Dispatcher

```php
use WordPress\AiClient\AiClient;
use YourNamespace\YourEventDispatcher;

// Set a PSR-14 compatible event dispatcher
$dispatcher = new YourEventDispatcher();
AiClient::setEventDispatcher($dispatcher);
```

### Available Events

#### BeforeGenerateResultEvent

Dispatched before sending a prompt to the model:

```php
use WordPress\AiClient\Events\BeforeGenerateResultEvent;

class LoggingListener
{
    public function onBeforeGenerate(BeforeGenerateResultEvent $event): void
    {
        $model = $event->getModel();
        $messages = $event->getMessages();
        $capability = $event->getCapability();
        
        // Log the request
        error_log("Sending prompt to model: " . $model->metadata()->getId());
        error_log("Capability: " . $capability->value);
        error_log("Message count: " . count($messages));
    }
}
```

#### AfterGenerateResultEvent

Dispatched after receiving a response:

```php
use WordPress\AiClient\Events\AfterGenerateResultEvent;

class MetricsListener
{
    public function onAfterGenerate(AfterGenerateResultEvent $event): void
    {
        $result = $event->getResult();
        $usage = $result->getTokenUsage();
        
        // Track token usage
        $this->recordMetric('prompt_tokens', $usage->getPromptTokens());
        $this->recordMetric('completion_tokens', $usage->getCompletionTokens());
        $this->recordMetric('total_tokens', $usage->getTotalTokens());
    }
    
    private function recordMetric(string $name, int $value): void
    {
        // Your metrics implementation
    }
}
```

### Event Listener Registration

Register your listeners with your PSR-14 dispatcher:

```php
$dispatcher->addListener(
    BeforeGenerateResultEvent::class,
    [new LoggingListener(), 'onBeforeGenerate']
);

$dispatcher->addListener(
    AfterGenerateResultEvent::class,
    [new MetricsListener(), 'onAfterGenerate']
);

AiClient::setEventDispatcher($dispatcher);
```

## Environment Variables

### Provider API Keys

| Provider | Environment Variable | Description |
|----------|---------------------|-------------|
| Google | `GOOGLE_API_KEY` | Google AI Studio API key |
| OpenAI | `OPENAI_API_KEY` | OpenAI platform API key |
| Anthropic | `ANTHROPIC_API_KEY` | Anthropic console API key |

### Custom Environment Variables

For custom providers, you can use any environment variable naming convention you prefer.

### Loading from .env Files

If using a package like `vlucas/phpdotenv`:

```php
// Load environment variables
$dotenv = Dotenv\Dotenv::createImmutable(__DIR__);
$dotenv->load();

// Now API keys are available
$text = AiClient::prompt('Hello!')
    ->generateText();
```

### Runtime Configuration

You can also set environment variables at runtime:

```php
putenv('GOOGLE_API_KEY=your-key-here');

$text = AiClient::prompt('Hello!')
    ->usingProvider('google')
    ->generateText();
```

## Model Preferences

Guide model selection when using automatic discovery:

```php
use WordPress\AiClient\ProviderImplementations\Google\GoogleProvider;
use WordPress\AiClient\ProviderImplementations\OpenAi\OpenAiProvider;

$text = AiClient::prompt('Write a story.')
    ->usingModelPreference(
        [GoogleProvider::class, 'gemini-2.0-flash-exp'],
        'openai',
        [OpenAiProvider::class, 'gpt-4o']
    )
    ->generateText();
```

The system will try models in the order specified:
1. Google's gemini-2.0-flash-exp
2. Any OpenAI model that supports the capability
3. OpenAI's gpt-4o

## Next Steps

- Learn about [Advanced Features](./ADVANCED.md) like custom providers and operations
- See the [API Reference](./API_REFERENCE.md) for detailed method documentation
- Review [Usage Examples](./USAGE.md) for common patterns
