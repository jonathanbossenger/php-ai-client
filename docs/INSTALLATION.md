# Installation

This guide explains how to install the PHP AI Client in your project.

## Requirements

- **PHP**: 7.4 or higher
- **Composer**: For dependency management
- **Extensions**: `ext-json`

## Installation via Composer

Install the package using Composer:

```bash
composer require wordpress/php-ai-client
```

## HTTP Client Requirement

The PHP AI Client uses PSR-18 HTTP clients for making API requests. You need to have a compatible HTTP client installed. The library uses [HTTPlug Discovery](https://docs.php-http.org/en/latest/discovery.html) to automatically detect available HTTP clients.

### Recommended HTTP Clients

If you don't already have an HTTP client, install one of these:

**Guzzle** (most popular):
```bash
composer require guzzlehttp/guzzle
```

**Symfony HTTP Client**:
```bash
composer require symfony/http-client
composer require nyholm/psr7
```

**cURL Client**:
```bash
composer require php-http/curl-client
composer require nyholm/psr7
```

### Automatic Discovery

The library will automatically discover and use any installed PSR-18 compatible HTTP client. You don't need to configure anything unless you want to use a specific client.

## Provider API Keys

To use the AI client, you need API keys from the AI providers you want to use. The library supports multiple providers:

### Google Gemini

1. Get an API key from [Google AI Studio](https://aistudio.google.com/app/apikey)
2. Set the environment variable:
   ```bash
   export GOOGLE_API_KEY="your-api-key-here"
   ```

### OpenAI

1. Get an API key from [OpenAI Platform](https://platform.openai.com/api-keys)
2. Set the environment variable:
   ```bash
   export OPENAI_API_KEY="your-api-key-here"
   ```

### Anthropic

1. Get an API key from [Anthropic Console](https://console.anthropic.com/)
2. Set the environment variable:
   ```bash
   export ANTHROPIC_API_KEY="your-api-key-here"
   ```

## Verifying Installation

Create a simple test script to verify your installation:

```php
<?php

require_once 'vendor/autoload.php';

use WordPress\AiClient\AiClient;

try {
    $text = AiClient::prompt('Say hello!')
        ->generateText();
    
    echo "Success! Response: " . $text . PHP_EOL;
} catch (Exception $e) {
    echo "Error: " . $e->getMessage() . PHP_EOL;
}
```

Run the script:

```bash
php test.php
```

If you see a response from the AI, your installation is working correctly!

## Troubleshooting

### "No HTTP client found"

If you get an error about no HTTP client being found:
1. Install one of the recommended HTTP clients listed above
2. Run `composer dump-autoload` to refresh the autoloader

### "No provider configured"

If you get an error about no provider being configured:
1. Make sure you've set the appropriate environment variable for your provider
2. Verify the API key is correct and has the necessary permissions

### PSR-17 Factory Issues

If you encounter issues with HTTP message factories, install:
```bash
composer require nyholm/psr7
```

## Next Steps

- Read the [Usage Guide](./USAGE.md) to learn how to use the client
- See [Configuration](./CONFIGURATION.md) for advanced configuration options
- Check the [Architecture](./ARCHITECTURE.md) document for technical details
