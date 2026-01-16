# Usage Guide

This guide covers common usage patterns for the PHP AI Client.

## Table of Contents

- [Basic Concepts](#basic-concepts)
- [Text Generation](#text-generation)
- [Image Generation](#image-generation)
- [Model Selection](#model-selection)
- [Configuration Options](#configuration-options)
- [Working with Messages](#working-with-messages)
- [Multimodal Input](#multimodal-input)
- [Error Handling](#error-handling)

## Basic Concepts

The PHP AI Client provides two APIs:

1. **Fluent API** (Recommended): Easy-to-read chained method calls
2. **Traditional API**: Array-based configuration for WordPress-style usage

Both APIs provide the same functionality, but the fluent API is generally more readable.

## Text Generation

### Simple Text Generation

Generate text using any available model:

```php
use WordPress\AiClient\AiClient;

$text = AiClient::prompt('Write a haiku about PHP.')
    ->generateText();

echo $text;
```

### Text Generation with Specific Provider

Use a specific provider's models:

```php
$text = AiClient::prompt('Explain how PHP works.')
    ->usingProvider('openai')
    ->generateText();
```

### Text Generation with Specific Model

Use a specific model from a provider:

```php
use WordPress\AiClient\ProviderImplementations\Google\GoogleProvider;

$text = AiClient::prompt('What is machine learning?')
    ->usingModel(GoogleProvider::model('gemini-2.5-flash'))
    ->generateText();
```

### Multiple Candidates

Generate multiple response options:

```php
$texts = AiClient::prompt('Write a tagline for a coffee shop.')
    ->generateTexts(3); // Generate 3 different options

foreach ($texts as $i => $text) {
    echo "Option " . ($i + 1) . ": " . $text . PHP_EOL;
}
```

## Image Generation

### Generate an Image

```php
$imageFile = AiClient::prompt('A serene mountain landscape at sunset')
    ->generateImage();

// Get base64 data
$base64Data = $imageFile->getBase64Data();

// Get URL (if available)
$url = $imageFile->getUrl();

// Get MIME type
$mimeType = $imageFile->getMimeType();
```

### Generate Multiple Images

```php
$images = AiClient::prompt('A cute robot mascot')
    ->generateImages(4);

foreach ($images as $i => $image) {
    // Save each image
    file_put_contents(
        "robot_{$i}.png",
        base64_decode($image->getBase64Data())
    );
}
```

## Model Selection

The client provides three ways to select models:

### 1. Automatic Discovery (Default)

The system automatically selects an appropriate model:

```php
// Automatically picks a text generation model
$text = AiClient::prompt('Hello!')
    ->generateText();
```

### 2. Provider Selection

Select a provider and let it choose the best model:

```php
$text = AiClient::prompt('Hello!')
    ->usingProvider('anthropic')
    ->generateText();
```

### 3. Specific Model Selection

Use a specific model instance:

```php
use WordPress\AiClient\ProviderImplementations\OpenAi\OpenAiProvider;

$model = OpenAiProvider::model('gpt-4o');

$text = AiClient::prompt('Explain quantum computing')
    ->usingModel($model)
    ->generateText();
```

## Configuration Options

### Temperature

Control randomness (0.0 = deterministic, 1.0 = creative):

```php
$text = AiClient::prompt('Write a story about a dragon.')
    ->usingTemperature(0.9) // More creative
    ->generateText();
```

### Max Tokens

Limit the response length:

```php
$text = AiClient::prompt('Summarize the history of computers.')
    ->usingMaxTokens(150)
    ->generateText();
```

### System Instructions

Provide system-level instructions to guide the model:

```php
$text = AiClient::prompt('Explain photosynthesis.')
    ->usingSystemInstruction('You are a biology teacher explaining concepts to high school students.')
    ->generateText();
```

### Top P (Nucleus Sampling)

Control diversity via nucleus sampling:

```php
$text = AiClient::prompt('Generate a product name.')
    ->usingTopP(0.9)
    ->generateText();
```

### Top K

Limit the number of tokens considered for each step:

```php
$text = AiClient::prompt('Complete this sentence: The best part about coding is')
    ->usingTopK(40)
    ->generateText();
```

### Combining Options

Chain multiple configuration methods:

```php
$text = AiClient::prompt('Write a poem about the ocean.')
    ->usingTemperature(0.8)
    ->usingMaxTokens(200)
    ->usingTopP(0.95)
    ->usingSystemInstruction('You are a professional poet.')
    ->generateText();
```

## Working with Messages

### Chat History

Provide conversation context:

```php
use WordPress\AiClient\Messages\DTO\UserMessage;
use WordPress\AiClient\Messages\DTO\ModelMessage;

$text = AiClient::prompt()
    ->withHistory(
        new UserMessage('What is WordPress?'),
        new ModelMessage('WordPress is a popular content management system.'),
        new UserMessage('Who created it?')
    )
    ->generateText();
```

### Building Complex Messages

Use the message builder for more control:

```php
$message = AiClient::message()
    ->usingUserRole()
    ->withText('Analyze this image:')
    ->withFile('/path/to/image.jpg', 'image/jpeg')
    ->get();

$text = AiClient::prompt()
    ->withHistory($message)
    ->generateText();
```

## Multimodal Input

### Text with Image Input

```php
// From file path
$text = AiClient::prompt('What is in this image?')
    ->withFile('/path/to/image.jpg', 'image/jpeg')
    ->generateText();

// From base64 data
$text = AiClient::prompt('Describe this image.')
    ->withInlineImage($base64Data, 'image/png')
    ->generateText();

// From URL
$text = AiClient::prompt('What do you see?')
    ->withFileUrl('https://example.com/image.jpg', 'image/jpeg')
    ->generateText();
```

### Multiple Images

```php
$text = AiClient::prompt('Compare these two images:')
    ->withFile('/path/to/image1.jpg', 'image/jpeg')
    ->withFile('/path/to/image2.jpg', 'image/jpeg')
    ->generateText();
```

## Error Handling

Always wrap AI operations in try-catch blocks:

```php
use WordPress\AiClient\AiClient;
use WordPress\AiClient\Common\Exception\InvalidArgumentException;
use WordPress\AiClient\Providers\Http\Exception\ResponseException;

try {
    $text = AiClient::prompt('Write a story.')
        ->generateText();
    
    echo $text;
} catch (InvalidArgumentException $e) {
    // Handle invalid arguments (bad configuration, etc.)
    echo "Invalid argument: " . $e->getMessage();
} catch (ResponseException $e) {
    // Handle API errors (rate limits, authentication, etc.)
    echo "API error: " . $e->getMessage();
} catch (Exception $e) {
    // Handle other errors
    echo "Error: " . $e->getMessage();
}
```

### Common Exceptions

- **InvalidArgumentException**: Invalid configuration or parameters
- **ResponseException**: API request failures (network, authentication, rate limits)
- **RuntimeException**: Unexpected runtime errors

### Checking Provider Availability

Before making requests, check if a provider is configured:

```php
use WordPress\AiClient\AiClient;
use WordPress\AiClient\ProviderImplementations\Google\GoogleProvider;

$availability = GoogleProvider::availability();

if ($availability->isConfigured()) {
    $text = AiClient::prompt('Hello!')
        ->usingProvider('google')
        ->generateText();
} else {
    echo "Google provider is not configured. Please set GOOGLE_API_KEY.";
}
```

## Working with Results

### Full Result Object

Get the complete result for more detailed information:

```php
$result = AiClient::prompt('Explain AI.')
    ->generateTextResult();

// Get the text
$text = $result->toText();

// Get token usage
$usage = $result->getTokenUsage();
echo "Prompt tokens: " . $usage->getPromptTokens() . PHP_EOL;
echo "Completion tokens: " . $usage->getCompletionTokens() . PHP_EOL;
echo "Total tokens: " . $usage->getTotalTokens() . PHP_EOL;

// Get candidates (multiple response options)
$candidates = $result->getCandidates();
foreach ($candidates as $candidate) {
    echo "Finish reason: " . $candidate->getFinishReason()->value . PHP_EOL;
    echo "Text: " . $candidate->getMessage()->getParts()[0]->getText() . PHP_EOL;
}

// Get provider metadata
$providerMeta = $result->getProviderMetadata();
$modelMeta = $result->getModelMetadata();
echo "Provider: " . $providerMeta->getName() . PHP_EOL;
echo "Model: " . $modelMeta->getName() . PHP_EOL;
```

## Next Steps

- Explore [Configuration](./CONFIGURATION.md) for advanced configuration
- Learn about [Advanced Features](./ADVANCED.md) like events and custom providers
- See the [API Reference](./API_REFERENCE.md) for detailed method documentation
