# PHP AI Client Documentation

Welcome to the PHP AI Client documentation! This directory contains comprehensive guides to help you install, configure, and use the PHP AI Client SDK.

## Documentation Structure

### Getting Started

1. **[Installation](./INSTALLATION.md)** - Install the SDK and set up your environment
   - System requirements
   - Installing via Composer
   - HTTP client setup
   - API key configuration
   - Troubleshooting

2. **[Usage Guide](./USAGE.md)** - Learn how to use the client
   - Basic concepts and APIs
   - Text generation
   - Image generation
   - Model selection
   - Configuration options
   - Working with messages
   - Multimodal input
   - Error handling

### Configuration & Advanced Usage

3. **[Configuration](./CONFIGURATION.md)** - Configure the client and providers
   - Provider configuration
   - Authentication methods
   - Model configuration
   - HTTP client configuration
   - Event dispatchers
   - Environment variables

4. **[Advanced Features](./ADVANCED.md)** - Extend and customize the SDK
   - Creating custom providers
   - Long-running operations
   - Streaming responses
   - Model discovery
   - Custom HTTP transport
   - Function calling
   - Provider metadata
   - Extending the SDK

### Reference Documentation

5. **[API Reference](./API_REFERENCE.md)** - Detailed API documentation
   - AiClient class
   - PromptBuilder class
   - MessageBuilder class
   - Results and DTOs
   - Messages and Files
   - Tools and Functions
   - Providers and Models
   - Events and Exceptions

### Project Information

6. **[Requirements](./REQUIREMENTS.md)** - Project requirements and objectives
   - Target audiences (extenders vs implementers)
   - Architecture requirements
   - Out of scope features

7. **[Architecture](./ARCHITECTURE.md)** - Technical architecture details
   - High-level API design
   - Code examples
   - Class diagrams
   - HTTP communication layer

8. **[Glossary](./GLOSSARY.md)** - Terminology and definitions
   - Key terms explained
   - Project-specific vocabulary

## Quick Links

### For New Users

Start here if you're new to the PHP AI Client:

1. Read [Installation](./INSTALLATION.md) to set up the SDK
2. Follow the [Usage Guide](./USAGE.md) to learn basic operations
3. Check the [API Reference](./API_REFERENCE.md) when you need details

### For Advanced Users

Explore these if you're familiar with the basics:

1. [Configuration](./CONFIGURATION.md) - Customize the client behavior
2. [Advanced Features](./ADVANCED.md) - Create custom providers and extend functionality
3. [Architecture](./ARCHITECTURE.md) - Understand the system design

### For Contributors

Read these to understand the project structure:

1. [Requirements](./REQUIREMENTS.md) - Project goals and constraints
2. [Architecture](./ARCHITECTURE.md) - System design and patterns
3. [Glossary](./GLOSSARY.md) - Common terminology
4. [Contributing Guide](../CONTRIBUTING.md) - Coding standards and guidelines

## Common Use Cases

### Text Generation

Generate text with any available model:

```php
use WordPress\AiClient\AiClient;

$text = AiClient::prompt('Write a haiku about PHP.')
    ->generateText();
```

[Learn more →](./USAGE.md#text-generation)

### Image Generation

Generate images from text descriptions:

```php
$image = AiClient::prompt('A serene mountain landscape')
    ->generateImage();
```

[Learn more →](./USAGE.md#image-generation)

### Multimodal Input

Process images with text:

```php
$text = AiClient::prompt('What is in this image?')
    ->withFile('/path/to/image.jpg', 'image/jpeg')
    ->generateText();
```

[Learn more →](./USAGE.md#multimodal-input)

### Chat with History

Maintain conversation context:

```php
use WordPress\AiClient\Messages\DTO\UserMessage;
use WordPress\AiClient\Messages\DTO\ModelMessage;

$text = AiClient::prompt()
    ->withHistory(
        new UserMessage('What is WordPress?'),
        new ModelMessage('WordPress is a content management system.'),
        new UserMessage('Who created it?')
    )
    ->generateText();
```

[Learn more →](./USAGE.md#chat-history)

### Model Selection

Choose specific providers or models:

```php
use WordPress\AiClient\ProviderImplementations\OpenAi\OpenAiProvider;

$text = AiClient::prompt('Explain quantum computing')
    ->usingModel(OpenAiProvider::model('gpt-4o'))
    ->generateText();
```

[Learn more →](./USAGE.md#model-selection)

### Configuration Options

Customize model behavior:

```php
$text = AiClient::prompt('Write a creative story.')
    ->usingTemperature(0.9)
    ->usingMaxTokens(500)
    ->usingSystemInstruction('You are a creative writer.')
    ->generateText();
```

[Learn more →](./USAGE.md#configuration-options)

## API Overview

The PHP AI Client provides two complementary APIs:

### Fluent API (Recommended)

Easy-to-read method chaining for implementers:

```php
$result = AiClient::prompt('Hello!')
    ->usingProvider('google')
    ->usingTemperature(0.7)
    ->generateTextResult();
```

### Traditional API

WordPress-style array-based configuration:

```php
$result = AiClient::generateTextResult(
    'Hello!',
    GoogleProvider::model('gemini-2.5-flash')
);
```

Both APIs provide the same functionality. Choose the one that fits your coding style.

## Supported Providers

The SDK includes built-in support for:

- **Google** - Gemini models for text and multimodal generation
- **OpenAI** - GPT models for text, images, and more
- **Anthropic** - Claude models for advanced text generation

You can also [create custom providers](./ADVANCED.md#custom-providers) to integrate other services.

## Key Features

- **Provider Agnostic** - Use any AI provider with a uniform API
- **Automatic Model Discovery** - Let the SDK choose the best model
- **Multimodal Support** - Text, images, audio, video, and more
- **Streaming Responses** - Stream text as it's generated
- **Event System** - Hook into the generation lifecycle
- **Type Safe** - Full type hints for PHP 7.4+
- **PSR Compliant** - Uses PSR-7, PSR-17, PSR-18, and PSR-14
- **Extensible** - Easy to add custom providers and models

## Requirements

- **PHP**: 7.4 or higher
- **Composer**: For dependency management
- **HTTP Client**: Any PSR-18 compatible client (auto-discovered)
- **API Keys**: From your chosen AI provider(s)

See [Installation](./INSTALLATION.md) for detailed setup instructions.

## Getting Help

### Documentation

- Check the relevant guide in this directory
- Search the [API Reference](./API_REFERENCE.md) for specific methods
- Review [Architecture](./ARCHITECTURE.md) for technical details

### Examples

- See code examples throughout the [Usage Guide](./USAGE.md)
- Check the [CLI script](../cli.php) for a working example
- Look at unit tests in the `tests/` directory

### Contributing

- Read the [Contributing Guide](../CONTRIBUTING.md) for coding standards
- Review the [Architecture](./ARCHITECTURE.md) to understand the design
- Check [Requirements](./REQUIREMENTS.md) for project goals

### Issues

- Report bugs on [GitHub Issues](https://github.com/WordPress/php-ai-client/issues)
- Ask questions in discussions
- Check existing issues before creating new ones

## License

This project is licensed under GPL-2.0-or-later. See [LICENSE.md](../LICENSE.md) for details.

## Project Links

- **GitHub**: https://github.com/WordPress/php-ai-client
- **WordPress AI Team**: https://make.wordpress.org/ai/
- **AI Building Blocks**: https://make.wordpress.org/ai/2025/07/17/ai-building-blocks

---

**Ready to get started?** Head over to [Installation](./INSTALLATION.md) to set up the SDK!
