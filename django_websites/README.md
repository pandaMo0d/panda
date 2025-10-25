# Django Websites

This directory contains all Django web application projects.

## Project Structure

Each Django website should follow this recommended structure:

```
project_name/
├── README.md              # Project-specific documentation
├── requirements.txt       # Python dependencies
├── .env.example          # Example environment variables
├── manage.py             # Django management script
├── project_name/         # Main project directory
│   ├── __init__.py
│   ├── settings.py       # Project settings
│   ├── urls.py           # URL configuration
│   ├── wsgi.py           # WSGI configuration
│   └── asgi.py           # ASGI configuration
├── apps/                 # Django applications
│   └── app_name/
│       ├── __init__.py
│       ├── models.py
│       ├── views.py
│       ├── urls.py
│       ├── admin.py
│       ├── apps.py
│       └── tests.py
├── static/               # Static files (CSS, JS, images)
├── media/                # User-uploaded files
├── templates/            # HTML templates
└── tests/                # Additional tests
```

## Getting Started

### Creating a New Django Project

1. Create a new directory and set up virtual environment:
   ```bash
   python -m venv venv
   source venv/bin/activate  # On Windows: venv\Scripts\activate
   ```

2. Install Django:
   ```bash
   pip install django
   pip freeze > requirements.txt
   ```

3. Create a new Django project:
   ```bash
   django-admin startproject project_name .
   ```

4. Create your first app:
   ```bash
   python manage.py startapp app_name
   ```

5. Set up your database:
   ```bash
   python manage.py migrate
   ```

6. Create a superuser:
   ```bash
   python manage.py createsuperuser
   ```

7. Run the development server:
   ```bash
   python manage.py runserver
   ```

### Common Dependencies

Most Django projects will need:
- `Django` - Web framework
- `python-dotenv` - Environment variable management
- `psycopg2-binary` - PostgreSQL adapter (if using PostgreSQL)
- `pillow` - Image processing library
- `django-cors-headers` - CORS handling
- `djangorestframework` - REST API (optional)
- `celery` - Task queue (optional)
- `redis` - Caching and task queue backend (optional)

## Best Practices

1. **Security**: 
   - Never commit `SECRET_KEY` or database credentials
   - Use environment variables for sensitive data
   - Keep `DEBUG = False` in production

2. **Database**:
   - Use PostgreSQL for production
   - Run migrations regularly
   - Keep migration files in version control

3. **Static Files**:
   - Use `collectstatic` for production
   - Configure proper static/media file serving

4. **Testing**:
   - Write tests for models, views, and forms
   - Use `python manage.py test` to run tests

5. **Documentation**:
   - Document API endpoints if building REST APIs
   - Keep README updated with setup instructions

6. **Code Organization**:
   - Follow Django's app-based architecture
   - Keep apps focused and reusable
   - Use Django's built-in features when possible

## Environment Variables

Create a `.env` file (copy from `.env.example`) with:
```
SECRET_KEY=your-secret-key-here
DEBUG=True
DATABASE_URL=postgresql://user:password@localhost:5432/dbname
ALLOWED_HOSTS=localhost,127.0.0.1
```

## Resources

- [Django Documentation](https://docs.djangoproject.com/)
- [Django Best Practices](https://django-best-practices.readthedocs.io/)
- [Django REST Framework](https://www.django-rest-framework.org/)
- [Two Scoops of Django](https://www.feldroy.com/books/two-scoops-of-django-3-x)
