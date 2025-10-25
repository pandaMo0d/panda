# Coding Best Practices

This document outlines coding standards and best practices for projects in this repository.

## General Python Best Practices

### Code Style

Follow [PEP 8](https://peps.python.org/pep-0008/) style guide:

```python
# Good
def calculate_total_price(items, tax_rate):
    """Calculate total price including tax."""
    subtotal = sum(item.price for item in items)
    total = subtotal * (1 + tax_rate)
    return total

# Bad
def calc(i,t):
    s=sum(x.price for x in i)
    return s*(1+t)
```

### Naming Conventions

- **Variables and functions**: `snake_case`
- **Classes**: `PascalCase`
- **Constants**: `UPPER_SNAKE_CASE`
- **Private methods**: `_leading_underscore`

```python
# Good examples
MAX_RETRY_COUNT = 3
user_name = "John"

class UserManager:
    def get_user_by_id(self, user_id):
        return self._fetch_from_database(user_id)
    
    def _fetch_from_database(self, user_id):
        # Private method
        pass
```

### Documentation

Use docstrings for modules, classes, and functions:

```python
def process_payment(amount, currency="USD"):
    """
    Process a payment transaction.
    
    Args:
        amount (float): The payment amount
        currency (str, optional): Currency code. Defaults to "USD".
    
    Returns:
        dict: Payment result with status and transaction_id
    
    Raises:
        ValueError: If amount is negative
        PaymentError: If payment processing fails
    """
    if amount < 0:
        raise ValueError("Amount cannot be negative")
    # ... processing logic
```

### Error Handling

Use specific exceptions and proper error handling:

```python
# Good
try:
    result = process_payment(amount)
except ValueError as e:
    logger.error(f"Invalid payment amount: {e}")
    raise
except PaymentError as e:
    logger.error(f"Payment failed: {e}")
    # Handle payment failure
    return {"status": "failed", "error": str(e)}

# Bad
try:
    result = process_payment(amount)
except:  # Too broad, catches everything
    pass  # Silently ignoring errors
```

### Logging

Use proper logging instead of print statements:

```python
import logging

logger = logging.getLogger(__name__)

# Good
logger.info("Processing order %s", order_id)
logger.warning("Low stock for product %s", product_id)
logger.error("Failed to connect to database", exc_info=True)

# Bad
print("Processing order", order_id)
```

## Telegram Bot Best Practices

### Bot Structure

Organize handlers logically:

```python
from telegram import Update
from telegram.ext import Application, CommandHandler, MessageHandler, filters

# Command handlers
async def start(update: Update, context):
    """Handle /start command."""
    await update.message.reply_text("Welcome!")

async def help_command(update: Update, context):
    """Handle /help command."""
    await update.message.reply_text("Available commands: /start, /help")

# Message handlers
async def echo(update: Update, context):
    """Echo user messages."""
    await update.message.reply_text(update.message.text)

# Main function
def main():
    """Start the bot."""
    application = Application.builder().token(TOKEN).build()
    
    # Register handlers
    application.add_handler(CommandHandler("start", start))
    application.add_handler(CommandHandler("help", help_command))
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, echo))
    
    # Run the bot
    application.run_polling()

if __name__ == '__main__':
    main()
```

### Configuration Management

Use environment variables for sensitive data:

```python
import os
from dotenv import load_dotenv

load_dotenv()

class Config:
    """Bot configuration."""
    BOT_TOKEN = os.getenv('BOT_TOKEN')
    DATABASE_URL = os.getenv('DATABASE_URL')
    ADMIN_IDS = [int(id) for id in os.getenv('ADMIN_IDS', '').split(',')]
    
    @classmethod
    def validate(cls):
        """Validate required configuration."""
        if not cls.BOT_TOKEN:
            raise ValueError("BOT_TOKEN is required")
```

### Error Handling in Bots

Implement error handlers:

```python
async def error_handler(update: Update, context):
    """Handle errors."""
    logger.error(f"Update {update} caused error {context.error}")
    
    if update and update.effective_message:
        await update.effective_message.reply_text(
            "An error occurred. Please try again later."
        )

# Register error handler
application.add_error_handler(error_handler)
```

### Rate Limiting

Implement rate limiting to prevent abuse:

```python
from functools import wraps
from datetime import datetime, timedelta

user_last_request = {}

def rate_limit(seconds=1):
    """Rate limit decorator."""
    def decorator(func):
        @wraps(func)
        async def wrapper(update, context, *args, **kwargs):
            user_id = update.effective_user.id
            now = datetime.now()
            
            if user_id in user_last_request:
                time_passed = now - user_last_request[user_id]
                if time_passed < timedelta(seconds=seconds):
                    await update.message.reply_text("Please wait before sending another request.")
                    return
            
            user_last_request[user_id] = now
            return await func(update, context, *args, **kwargs)
        return wrapper
    return decorator

@rate_limit(seconds=5)
async def expensive_operation(update, context):
    """Rate-limited command."""
    # ... expensive operation
    pass
```

## Django Best Practices

### Models

Write clean, well-documented models:

```python
from django.db import models
from django.core.validators import MinValueValidator

class Product(models.Model):
    """Product model."""
    
    name = models.CharField(max_length=200, db_index=True)
    description = models.TextField(blank=True)
    price = models.DecimalField(
        max_digits=10,
        decimal_places=2,
        validators=[MinValueValidator(0)]
    )
    stock = models.PositiveIntegerField(default=0)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)
    
    class Meta:
        ordering = ['-created_at']
        verbose_name = 'Product'
        verbose_name_plural = 'Products'
    
    def __str__(self):
        return self.name
    
    def is_in_stock(self):
        """Check if product is in stock."""
        return self.stock > 0
```

### Views

Use class-based views for common patterns:

```python
from django.views.generic import ListView, DetailView, CreateView
from django.contrib.auth.mixins import LoginRequiredMixin

class ProductListView(ListView):
    """List all products."""
    model = Product
    template_name = 'products/list.html'
    context_object_name = 'products'
    paginate_by = 20
    
    def get_queryset(self):
        """Filter products in stock."""
        return Product.objects.filter(stock__gt=0)

class ProductCreateView(LoginRequiredMixin, CreateView):
    """Create a new product."""
    model = Product
    fields = ['name', 'description', 'price', 'stock']
    template_name = 'products/form.html'
    success_url = '/products/'
```

### URL Patterns

Organize URLs clearly:

```python
from django.urls import path
from . import views

app_name = 'products'

urlpatterns = [
    path('', views.ProductListView.as_view(), name='list'),
    path('<int:pk>/', views.ProductDetailView.as_view(), name='detail'),
    path('create/', views.ProductCreateView.as_view(), name='create'),
    path('<int:pk>/update/', views.ProductUpdateView.as_view(), name='update'),
    path('<int:pk>/delete/', views.ProductDeleteView.as_view(), name='delete'),
]
```

### Settings Organization

Split settings for different environments:

```
settings/
├── __init__.py
├── base.py        # Common settings
├── development.py # Development settings
├── production.py  # Production settings
└── testing.py     # Test settings
```

```python
# base.py - Common settings
import os
from pathlib import Path

BASE_DIR = Path(__file__).resolve().parent.parent.parent

SECRET_KEY = os.getenv('SECRET_KEY')
DEBUG = False

INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    # ... more apps
]

# development.py
from .base import *

DEBUG = True
ALLOWED_HOSTS = ['localhost', '127.0.0.1']

# production.py
from .base import *

DEBUG = False
ALLOWED_HOSTS = os.getenv('ALLOWED_HOSTS', '').split(',')
```

### Templates

Use template inheritance:

```html
<!-- base.html -->
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}My Site{% endblock %}</title>
</head>
<body>
    <nav>
        {% block navigation %}{% endblock %}
    </nav>
    
    <main>
        {% block content %}{% endblock %}
    </main>
    
    <footer>
        {% block footer %}{% endblock %}
    </footer>
</body>
</html>

<!-- product_list.html -->
{% extends "base.html" %}

{% block title %}Products - {{ block.super }}{% endblock %}

{% block content %}
    <h1>Products</h1>
    {% for product in products %}
        <div class="product">
            <h2>{{ product.name }}</h2>
            <p>{{ product.description }}</p>
            <p>${{ product.price }}</p>
        </div>
    {% endfor %}
{% endblock %}
```

## Testing Best Practices

### Write Meaningful Tests

```python
import unittest
from telegram import Update, User, Message, Chat
from unittest.mock import MagicMock

class TestBotHandlers(unittest.TestCase):
    """Test bot command handlers."""
    
    def setUp(self):
        """Set up test fixtures."""
        self.user = User(id=1, first_name="Test", is_bot=False)
        self.chat = Chat(id=1, type="private")
    
    def test_start_command(self):
        """Test /start command sends welcome message."""
        # Arrange
        message = Message(
            message_id=1,
            date=None,
            chat=self.chat,
            from_user=self.user,
            text="/start"
        )
        update = Update(update_id=1, message=message)
        context = MagicMock()
        
        # Act
        result = start(update, context)
        
        # Assert
        self.assertTrue(message.reply_text.called)
        self.assertIn("Welcome", message.reply_text.call_args[0][0])
```

### Django Tests

```python
from django.test import TestCase, Client
from django.contrib.auth.models import User
from .models import Product

class ProductModelTest(TestCase):
    """Test Product model."""
    
    def setUp(self):
        """Create test product."""
        self.product = Product.objects.create(
            name="Test Product",
            price=10.99,
            stock=5
        )
    
    def test_product_is_in_stock(self):
        """Test is_in_stock method."""
        self.assertTrue(self.product.is_in_stock())
        
        self.product.stock = 0
        self.product.save()
        self.assertFalse(self.product.is_in_stock())
    
    def test_product_str(self):
        """Test string representation."""
        self.assertEqual(str(self.product), "Test Product")

class ProductViewTest(TestCase):
    """Test Product views."""
    
    def setUp(self):
        """Set up test client and data."""
        self.client = Client()
        self.product = Product.objects.create(
            name="Test Product",
            price=10.99,
            stock=5
        )
    
    def test_product_list_view(self):
        """Test product list view returns 200."""
        response = self.client.get('/products/')
        self.assertEqual(response.status_code, 200)
        self.assertContains(response, "Test Product")
```

## Security Best Practices

### Never Commit Secrets

```python
# Good - Use environment variables
import os
API_KEY = os.getenv('API_KEY')

# Bad - Hardcoded secrets
API_KEY = "sk_live_1234567890abcdef"  # NEVER DO THIS
```

### Input Validation

```python
# Good - Validate and sanitize input
def process_user_input(user_input):
    """Process user input safely."""
    if not isinstance(user_input, str):
        raise TypeError("Input must be string")
    
    if len(user_input) > 1000:
        raise ValueError("Input too long")
    
    # Sanitize input
    cleaned_input = user_input.strip()
    return cleaned_input

# Bad - No validation
def process_user_input(user_input):
    return user_input  # Dangerous!
```

### SQL Injection Prevention

```python
# Good - Use Django ORM or parameterized queries
products = Product.objects.filter(name=user_input)

# Bad - String concatenation in raw SQL
cursor.execute(f"SELECT * FROM products WHERE name = '{user_input}'")  # NEVER DO THIS
```

## Performance Best Practices

### Database Queries

```python
# Good - Use select_related for foreign keys
products = Product.objects.select_related('category').all()

# Good - Use prefetch_related for many-to-many
products = Product.objects.prefetch_related('tags').all()

# Bad - N+1 queries
products = Product.objects.all()
for product in products:
    print(product.category.name)  # Triggers query for each product
```

### Caching

```python
from django.core.cache import cache

def get_expensive_data(key):
    """Get data with caching."""
    data = cache.get(key)
    if data is None:
        data = expensive_database_query()
        cache.set(key, data, timeout=3600)  # Cache for 1 hour
    return data
```

## Version Control Best Practices

### Commit Messages

Use conventional commit format:

```
feat: add user authentication
fix: resolve payment processing bug
docs: update deployment guide
refactor: simplify product model
test: add tests for order processing
chore: update dependencies
```

### Branching Strategy

- `main` - Production-ready code
- `develop` - Development branch
- `feature/*` - Feature branches
- `bugfix/*` - Bug fix branches
- `hotfix/*` - Production hotfixes

## Resources

- [PEP 8 Style Guide](https://peps.python.org/pep-0008/)
- [Google Python Style Guide](https://google.github.io/styleguide/pyguide.html)
- [Django Best Practices](https://django-best-practices.readthedocs.io/)
- [The Zen of Python](https://peps.python.org/pep-0020/)
- [Clean Code in Python](https://github.com/zedr/clean-code-python)
