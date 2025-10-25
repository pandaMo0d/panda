# Telegram Bots

This directory contains all Telegram bot projects.

## Project Structure

Each bot should follow this recommended structure:

```
bot_name/
├── README.md              # Bot-specific documentation
├── requirements.txt       # Python dependencies
├── .env.example          # Example environment variables
├── bot.py                # Main bot file
├── config.py             # Configuration management
├── handlers/             # Message and command handlers
│   ├── __init__.py
│   ├── commands.py
│   └── messages.py
├── utils/                # Utility functions
│   ├── __init__.py
│   └── helpers.py
└── tests/                # Unit tests
    ├── __init__.py
    └── test_bot.py
```

## Getting Started

### Creating a New Bot

1. Create a new directory for your bot:
   ```bash
   mkdir my_bot
   cd my_bot
   ```

2. Set up a virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

3. Install required packages:
   ```bash
   pip install python-telegram-bot python-dotenv
   pip freeze > requirements.txt
   ```

4. Create your bot configuration:
   - Create a `.env` file with your bot token (copy from `.env.example`)
   - Never commit your `.env` file to version control

### Common Dependencies

Most Telegram bots will need:
- `python-telegram-bot` - Telegram Bot API wrapper
- `python-dotenv` - Environment variable management
- `requests` - HTTP library (optional)
- `aiohttp` - Async HTTP client (optional)

## Best Practices

1. **Security**: Never commit bot tokens or API keys
2. **Documentation**: Each bot should have a detailed README
3. **Environment Variables**: Use `.env` files for configuration
4. **Error Handling**: Implement proper error handling and logging
5. **Testing**: Write unit tests for your bot handlers
6. **Async**: Consider using async/await for better performance

## Resources

- [python-telegram-bot Documentation](https://docs.python-telegram-bot.org/)
- [Telegram Bot API](https://core.telegram.org/bots/api)
- [Creating a Telegram Bot](https://core.telegram.org/bots#creating-a-new-bot)
