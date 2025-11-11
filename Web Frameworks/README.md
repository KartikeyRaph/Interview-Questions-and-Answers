# Web Frameworks Interview Q&A

## Overview
This section covers Django, Flask, and FastAPI for interview preparation.

## Frameworks Covered

### Django
- Project Structure
- Models, Views, Templates (MVT)
- ORM & QuerySets
- Forms & Validation
- Authentication & Authorization
- Middleware
- Django Admin
- Testing

### Flask
- Application Factory Pattern
- Routing & Blueprints
- Jinja2 Templates
- Request/Response Handling
- Sessions & Cookies
- Extensions (SQLAlchemy, WTForms, etc.)
- Error Handling
- Testing

### FastAPI
- Async/Await
- Request & Response Models
- Dependency Injection
- Security & Authentication
- Automatic API Documentation
- Performance Optimization
- Middleware
- Testing

## Q&A

---

# DJANGO

---

## Q1: Explain Django's MVT (Model-View-Template) architecture.

**Answer:**

Django follows the **MVT (Model-View-Template)** architectural pattern, which is similar to MVC:

- **Model:** Database layer (ORM)
- **View:** Business logic layer (Python functions/classes)
- **Template:** Presentation layer (HTML with template tags)

```python
# Project Structure
myproject/
├── myproject/
│   ├── __init__.py
│   ├── settings.py      # Configuration
│   ├── urls.py          # URL routing
│   ├── wsgi.py          # WSGI config
│   └── asgi.py          # ASGI config
├── myapp/
│   ├── models.py        # Model
│   ├── views.py         # View
│   ├── urls.py          # App URLs
│   ├── templates/       # Template
│   ├── static/          # Static files
│   └── admin.py         # Admin config
└── manage.py

# 1. MODEL - Database layer
from django.db import models

class Product(models.Model):
    name = models.CharField(max_length=100)
    price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    
    class Meta:
        db_table = 'products'
        ordering = ['-created_at']

# 2. VIEW - Business logic
from django.shortcuts import render, get_object_or_404
from django.views import View
from django.views.generic import ListView, DetailView

# Function-based view
def product_list(request):
    products = Product.objects.all()
    return render(request, 'products/list.html', {'products': products})

# Class-based view
class ProductListView(ListView):
    model = Product
    template_name = 'products/list.html'
    context_object_name = 'products'
    paginate_by = 10

class ProductDetailView(DetailView):
    model = Product
    template_name = 'products/detail.html'
    context_object_name = 'product'

# 3. TEMPLATE - Presentation layer (HTML)
# products/list.html
"""
{% extends 'base.html' %}
{% load static %}

{% block content %}
<h1>Products</h1>
<div class="products">
    {% for product in products %}
        <div class="product-card">
            <h2>{{ product.name }}</h2>
            <p>Price: ${{ product.price }}</p>
            <a href="{% url 'product-detail' product.pk %}">View</a>
        </div>
    {% endfor %}
</div>
{% endblock %}
"""

# 4. URL ROUTING - Connects view to URL
# myapp/urls.py
from django.urls import path
from . import views

urlpatterns = [
    path('products/', views.ProductListView.as_view(), name='product-list'),
    path('products/<int:pk>/', views.ProductDetailView.as_view(), name='product-detail'),
]

# myproject/urls.py
from django.contrib import admin
from django.urls import path, include

urlpatterns = [
    path('admin/', admin.site.urls),
    path('app/', include('myapp.urls')),
]
```

**Request Flow:**
1. User requests URL → Django matches with URL pattern
2. Django calls corresponding **View**
3. View queries **Model** (database)
4. View renders **Template** with data
5. Response sent to user

---

## Q2: What is the Django ORM? Explain QuerySets and common operations.

**Answer:**

Django's **ORM (Object-Relational Mapping)** allows interaction with database using Python objects instead of raw SQL.

```python
from django.db import models
from django.db.models import Q, Count, Sum, Avg

# Define Models
class Author(models.Model):
    name = models.CharField(max_length=100)
    email = models.EmailField()
    created_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.name

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.ForeignKey(Author, on_delete=models.CASCADE, related_name='books')
    published_date = models.DateField()
    price = models.DecimalField(max_digits=5, decimal_places=2)
    
    class Meta:
        indexes = [models.Index(fields=['published_date'])]

# BASIC CRUD OPERATIONS

# Create
author = Author.objects.create(name='John Doe', email='john@example.com')
book = Book(title='Python Guide', author=author, published_date='2023-01-01', price=29.99)
book.save()

# Read (all)
all_authors = Author.objects.all()

# Read (single)
author = Author.objects.get(id=1)
author = Author.objects.get(name='John Doe')
try:
    author = Author.objects.get(id=999)
except Author.DoesNotExist:
    print("Not found")

# Read (with default)
author = Author.objects.filter(name='Unknown').first()

# Update
author.email = 'newemail@example.com'
author.save()

# Or bulk update
Author.objects.filter(name='John').update(email='john.new@example.com')

# Delete
author.delete()

# Or bulk delete
Author.objects.filter(id__gt=10).delete()

# QUERYSET FILTERING

# Exact match
books = Book.objects.filter(title='Python Guide')

# Case-insensitive match
books = Book.objects.filter(title__iexact='python guide')

# Contains
books = Book.objects.filter(title__icontains='python')

# Starts/Ends with
books = Book.objects.filter(title__startswith='Python')
books = Book.objects.filter(title__endswith='Guide')

# Range
from datetime import date
books = Book.objects.filter(published_date__range=['2020-01-01', '2023-12-31'])

# Comparison operators
books = Book.objects.filter(price__gt=20)      # Greater than
books = Book.objects.filter(price__gte=20)     # Greater than or equal
books = Book.objects.filter(price__lt=50)      # Less than
books = Book.objects.filter(price__lte=50)     # Less than or equal

# In a list
books = Book.objects.filter(id__in=[1, 2, 3])

# Null checks
books = Book.objects.filter(author__isnull=True)

# Complex queries with Q objects
books = Book.objects.filter(
    Q(title__contains='Python') | Q(author__name='John')
)

books = Book.objects.filter(
    Q(price__gt=20) & ~Q(author__name='Unknown')
)

# RELATIONSHIPS

# Foreign key access
book = Book.objects.first()
author_name = book.author.name

# Reverse relation
author = Author.objects.first()
author_books = author.books.all()

# Filtering through relationships
books = Book.objects.filter(author__name='John Doe')
authors = Author.objects.filter(books__price__gt=30)

# AGGREGATIONS
from django.db.models import Count, Sum, Avg, Max, Min

# Count
total_books = Book.objects.count()
books_per_author = Book.objects.values('author').annotate(count=Count('id'))

# Sum, Avg, Max, Min
total_price = Book.objects.aggregate(total=Sum('price'))
avg_price = Book.objects.aggregate(average=Avg('price'))
most_expensive = Book.objects.aggregate(max_price=Max('price'))

# Aggregation per author
authors = Author.objects.annotate(
    total_books=Count('books'),
    total_revenue=Sum('books__price'),
    avg_price=Avg('books__price')
)

for author in authors:
    print(f"{author.name}: {author.total_books} books")

# ORDERING
books = Book.objects.order_by('published_date')
books = Book.objects.order_by('-price')  # Descending
books = Book.objects.order_by('author__name', '-price')

# LIMITING & SLICING
first_5_books = Book.objects.all()[:5]
page_2 = Book.objects.all()[10:20]  # Offset 10, limit 10

# DISTINCT
unique_authors = Book.objects.values('author').distinct()

# OPTIMIZATION

# Select related (for one-to-many, many-to-one)
books = Book.objects.select_related('author')  # Single query

# Prefetch related (for many-to-many, reverse foreign key)
authors = Author.objects.prefetch_related('books')

# Only/Defer (select specific fields)
books = Book.objects.only('title', 'price')  # Load only these fields
books = Book.objects.defer('description')    # Exclude these fields

# EXISTS
has_books = Book.objects.filter(author__name='John').exists()

# VALUES/VALUES_LIST
titles = Book.objects.values_list('title', flat=True)
data = Book.objects.values('title', 'author__name')

# RAW SQL
from django.db import connection
with connection.cursor() as cursor:
    cursor.execute("SELECT * FROM book WHERE price > %s", [50])
    results = cursor.fetchall()
```

**Key Points:**
- QuerySets are lazy (evaluated on access)
- Chain methods for complex queries
- Use select_related() and prefetch_related() for optimization
- Avoid N+1 queries

---

## Q3: What are Django Forms? Explain ModelForm with validation.

**Answer:**

Django Forms handle HTML form generation, validation, and data processing.

```python
from django import forms
from django.core.exceptions import ValidationError
from .models import Book, Author

# 1. REGULAR FORM
class ContactForm(forms.Form):
    name = forms.CharField(
        max_length=100,
        widget=forms.TextInput(attrs={'class': 'form-control'})
    )
    email = forms.EmailField(
        widget=forms.EmailInput(attrs={'class': 'form-control'})
    )
    message = forms.CharField(
        widget=forms.Textarea(attrs={'rows': 5, 'class': 'form-control'})
    )
    subject = forms.ChoiceField(
        choices=[('support', 'Support'), ('feedback', 'Feedback')],
        widget=forms.Select(attrs={'class': 'form-control'})
    )

# 2. MODELFORM (automatically creates form from model)
class BookForm(forms.ModelForm):
    class Meta:
        model = Book
        fields = ['title', 'author', 'published_date', 'price']
        widgets = {
            'title': forms.TextInput(attrs={'class': 'form-control', 'placeholder': 'Enter title'}),
            'author': forms.Select(attrs={'class': 'form-control'}),
            'published_date': forms.DateInput(attrs={'type': 'date', 'class': 'form-control'}),
            'price': forms.NumberInput(attrs={'step': '0.01', 'class': 'form-control'}),
        }
        labels = {
            'title': 'Book Title',
            'published_date': 'Published Date'
        }

# 3. CUSTOM VALIDATION
class UserRegistrationForm(forms.Form):
    username = forms.CharField(max_length=50)
    password = forms.CharField(widget=forms.PasswordInput)
    password_confirm = forms.CharField(widget=forms.PasswordInput)
    email = forms.EmailField()
    
    # Field-level validation
    def clean_username(self):
        username = self.cleaned_data.get('username')
        if len(username) < 3:
            raise ValidationError('Username must be at least 3 characters')
        if User.objects.filter(username=username).exists():
            raise ValidationError('Username already exists')
        return username
    
    # Form-level validation
    def clean(self):
        cleaned_data = super().clean()
        password = cleaned_data.get('password')
        password_confirm = cleaned_data.get('password_confirm')
        
        if password != password_confirm:
            raise ValidationError('Passwords do not match')
        
        return cleaned_data

# 4. USING FORMS IN VIEWS
from django.shortcuts import render, redirect
from django.views import View
from django.views.generic import CreateView, UpdateView
from django.urls import reverse_lazy

# Function-based view
def create_book(request):
    if request.method == 'POST':
        form = BookForm(request.POST, request.FILES)
        if form.is_valid():
            book = form.save()  # Save to database
            return redirect('book-detail', pk=book.id)
    else:
        form = BookForm()
    return render(request, 'book_form.html', {'form': form})

# Class-based view
class BookCreateView(CreateView):
    model = Book
    form_class = BookForm
    template_name = 'book_form.html'
    success_url = reverse_lazy('book-list')
    
    def form_valid(self, form):
        # Custom logic before save
        form.instance.created_by = self.request.user
        return super().form_valid(form)

class BookUpdateView(UpdateView):
    model = Book
    form_class = BookForm
    template_name = 'book_form.html'
    success_url = reverse_lazy('book-list')

# 5. TEMPLATE RENDERING
"""
{% extends 'base.html' %}
{% load static %}

{% block content %}
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    
    <div class="form-group">
        {{ form.title.label_tag }}
        {{ form.title }}
        {% if form.title.errors %}
            <div class="error">{{ form.title.errors }}</div>
        {% endif %}
    </div>
    
    {% for field in form %}
        <div class="form-group">
            {{ field.label_tag }}
            {{ field }}
            {% if field.errors %}
                <div class="error">{{ field.errors }}</div>
            {% endif %}
        </div>
    {% endfor %}
    
    {% if form.non_field_errors %}
        <div class="error">{{ form.non_field_errors }}</div>
    {% endif %}
    
    <button type="submit" class="btn btn-primary">Submit</button>
</form>
{% endblock %}
"""

# 6. FORMSET (multiple forms)
from django.forms import formset_factory

BookFormSet = formset_factory(BookForm, extra=3, max_num=10)

def create_books(request):
    if request.method == 'POST':
        formset = BookFormSet(request.POST)
        if formset.is_valid():
            for form in formset:
                if form.cleaned_data:
                    form.save()
            return redirect('book-list')
    else:
        formset = BookFormSet()
    return render(request, 'book_formset.html', {'formset': formset})
```

**Key Features:**
- Auto-validation from model fields
- CSRF protection
- Custom validators
- Field widgets and attributes
- Error handling and display
- FormSets for multiple forms

---

## Q4: Explain Django Middleware and create custom middleware.

**Answer:**

**Middleware** are hooks/filters that process requests and responses globally in Django.

```python
# Django request/response cycle with middleware:
# Request → Middleware (process_request) → View → Middleware (process_response) → Response

# BUILT-IN MIDDLEWARE (in settings.py)
MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',
    'django.middleware.csrf.CsrfViewMiddleware',
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
]

# 1. CUSTOM MIDDLEWARE - Class-based
class CustomHeaderMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Code executed before view
        print(f"Request path: {request.path}")
        request.custom_attribute = "Custom Value"
        
        response = self.get_response(request)
        
        # Code executed after view
        response['X-Custom-Header'] = 'Custom Value'
        return response

# 2. LOGGING MIDDLEWARE
class LoggingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        import time
        import logging
        
        logger = logging.getLogger(__name__)
        start_time = time.time()
        
        response = self.get_response(request)
        
        duration = time.time() - start_time
        logger.info(
            f"Method: {request.method} | "
            f"Path: {request.path} | "
            f"Status: {response.status_code} | "
            f"Duration: {duration:.3f}s"
        )
        return response

# 3. AUTHENTICATION MIDDLEWARE
class CustomAuthMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        # Check for custom auth token
        token = request.META.get('HTTP_X_AUTH_TOKEN')
        if token:
            try:
                user = User.objects.get(auth_token=token)
                request.user = user
            except User.DoesNotExist:
                pass
        
        response = self.get_response(request)
        return response

# 4. EXCEPTION HANDLING MIDDLEWARE
class ErrorHandlingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        try:
            response = self.get_response(request)
        except Exception as e:
            # Log exception
            logger.error(f"Exception: {str(e)}", exc_info=True)
            # Return custom error response
            from django.http import JsonResponse
            return JsonResponse({'error': 'Internal Server Error'}, status=500)
        
        return response

# 5. CORS MIDDLEWARE
class CORSMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        response = self.get_response(request)
        
        # Add CORS headers
        response['Access-Control-Allow-Origin'] = '*'
        response['Access-Control-Allow-Methods'] = 'GET, POST, PUT, DELETE, OPTIONS'
        response['Access-Control-Allow-Headers'] = 'Content-Type, Authorization'
        
        if request.method == 'OPTIONS':
            return response
        
        return response

# 6. TIMING MIDDLEWARE
import time
class TimingMiddleware:
    def __init__(self, get_response):
        self.get_response = get_response
    
    def __call__(self, request):
        start_time = time.time()
        response = self.get_response(request)
        end_time = time.time()
        
        response['X-Process-Time'] = f"{end_time - start_time:.3f}"
        return response

# REGISTER MIDDLEWARE in settings.py
MIDDLEWARE = [
    # ... other middleware ...
    'myapp.middleware.CustomHeaderMiddleware',
    'myapp.middleware.LoggingMiddleware',
    'myapp.middleware.ErrorHandlingMiddleware',
]

# Middleware with process_request and process_response (Old style)
class OldStyleMiddleware:
    def process_request(self, request):
        # Executed before view
        print(f"Request: {request.path}")
        return None  # Return None to continue, or HttpResponse to short-circuit
    
    def process_view(self, request, view_func, view_args, view_kwargs):
        # Executed before view function
        return None
    
    def process_exception(self, request, exception):
        # Handle exceptions
        return None
    
    def process_template_response(self, request, response):
        # Process template response
        return response
    
    def process_response(self, request, response):
        # Executed after view
        return response
```

**Key Points:**
- Middleware executes in order of MIDDLEWARE list
- Each middleware has `__init__` and `__call__`
- Use for logging, authentication, error handling, CORS
- Raise Http404, PermissionDenied to short-circuit

---

## Q5: What is Django's authentication and authorization system?

**Answer:**

```python
# AUTHENTICATION & AUTHORIZATION

from django.contrib.auth.models import User
from django.contrib.auth.decorators import login_required, permission_required
from django.contrib.auth.mixins import LoginRequiredMixin, PermissionRequiredMixin
from django.views.generic import View
from django.shortcuts import redirect
from django.contrib.auth import authenticate, login, logout

# 1. USER CREATION & PASSWORD MANAGEMENT
# Create user
user = User.objects.create_user(
    username='johndoe',
    email='john@example.com',
    password='secure_password123'
)

# Create superuser
User.objects.create_superuser(
    username='admin',
    email='admin@example.com',
    password='admin_password123'
)

# Change password
user.set_password('new_password')
user.save()

# Check password
if user.check_password('new_password'):
    print("Password correct")

# 2. AUTHENTICATION
def login_view(request):
    if request.method == 'POST':
        username = request.POST['username']
        password = request.POST['password']
        
        # Authenticate user
        user = authenticate(request, username=username, password=password)
        
        if user is not None:
            login(request, user)
            return redirect('home')
        else:
            return render(request, 'login.html', {'error': 'Invalid credentials'})
    
    return render(request, 'login.html')

# Logout
def logout_view(request):
    logout(request)
    return redirect('home')

# 3. AUTHORIZATION - Function-based views
@login_required(login_url='login')
def dashboard(request):
    return render(request, 'dashboard.html')

@login_required
@permission_required('book.add_book', raise_exception=True)
def create_book(request):
    # Only users with 'add_book' permission
    return render(request, 'create_book.html')

@permission_required('book.delete_book')
def delete_book(request, pk):
    # Multiple permissions
    pass

# 4. AUTHORIZATION - Class-based views
class DashboardView(LoginRequiredMixin, View):
    login_url = 'login'
    redirect_field_name = 'next'
    
    def get(self, request):
        return render(request, 'dashboard.html')

class BookCreateView(PermissionRequiredMixin, CreateView):
    permission_required = 'book.add_book'
    raise_exception = True  # Raise 403 instead of redirect
    
    model = Book
    form_class = BookForm
    template_name = 'book_form.html'

class AdminOnlyView(PermissionRequiredMixin, View):
    permission_required = ('book.add_book', 'book.change_book', 'book.delete_book')
    
    def get(self, request):
        return render(request, 'admin.html')

# 5. CUSTOM PERMISSIONS
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    
    class Meta:
        permissions = [
            ('can_publish', 'Can publish books'),
            ('can_review', 'Can review books'),
        ]

# Check permission in view
if request.user.has_perm('book.can_publish'):
    # User can publish

# 6. GROUP-BASED PERMISSIONS
from django.contrib.auth.models import Group, Permission

# Create group
editors_group, created = Group.objects.get_or_create(name='Editors')

# Add permissions to group
permission = Permission.objects.get(codename='change_book')
editors_group.permissions.add(permission)

# Add user to group
user.groups.add(editors_group)

# Check if user in group
if user.groups.filter(name='Editors').exists():
    print("User is editor")

# 7. CUSTOM USER MODEL
from django.contrib.auth.models import AbstractUser

class CustomUser(AbstractUser):
    bio = models.TextField(blank=True)
    profile_picture = models.ImageField(upload_to='profiles/', null=True, blank=True)
    is_verified = models.BooleanField(default=False)
    
    def __str__(self):
        return self.username

# Add to settings.py
AUTH_USER_MODEL = 'myapp.CustomUser'

# 8. PERMISSION CHECKING IN TEMPLATES
"""
{% if user.is_authenticated %}
    Welcome, {{ user.username }}!
    
    {% if user.is_staff %}
        <a href="/admin/">Admin Panel</a>
    {% endif %}
    
    {% if user.has_perm 'book.add_book' %}
        <a href="/books/create/">Create Book</a>
    {% endif %}
    
    <a href="/logout/">Logout</a>
{% else %}
    <a href="/login/">Login</a>
{% endif %}
"""

# 9. PROFILE CACHING
class UserProfile(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile')
    bio = models.TextField()
    avatar = models.ImageField(upload_to='avatars/')

# Use profile
user.profile.bio

# 10. SESSION MANAGEMENT
def session_example(request):
    # Set session data
    request.session['user_id'] = request.user.id
    request.session['preferences'] = {'theme': 'dark'}
    
    # Get session data
    user_id = request.session.get('user_id')
    
    # Delete session data
    del request.session['user_id']
    
    # Clear all session data
    request.session.flush()
    
    return render(request, 'example.html')
```

**Key Points:**
- Use `LoginRequiredMixin` for CBV, `@login_required` for FBV
- `PermissionRequiredMixin` for permission checking
- Custom permissions for fine-grained access control
- Groups for managing multiple permissions
- Custom User model for additional fields
- Session data stored server-side

---

## Q6: How do you handle file uploads in Django?

**Answer:**

```python
from django.db import models
from django.core.validators import FileExtensionValidator
from PIL import Image

# MODEL WITH FILE FIELDS
class DocumentUpload(models.Model):
    FILE_CHOICES = [
        ('pdf', 'PDF Document'),
        ('doc', 'Word Document'),
        ('excel', 'Excel Sheet'),
    ]
    
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    title = models.CharField(max_length=200)
    file = models.FileField(upload_to='documents/%Y/%m/')
    file_type = models.CharField(max_length=10, choices=FILE_CHOICES)
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    def __str__(self):
        return self.title

class ProfilePicture(models.Model):
    user = models.OneToOneField(User, on_delete=models.CASCADE, related_name='profile_picture')
    image = models.ImageField(upload_to='profiles/')
    uploaded_at = models.DateTimeField(auto_now_add=True)
    
    def save(self, *args, **kwargs):
        super().save(*args, **kwargs)
        # Resize image on save
        img = Image.open(self.image.path)
        if img.height > 300 or img.width > 300:
            output_size = (300, 300)
            img.thumbnail(output_size)
            img.save(self.image.path)

# FORM WITH FILE UPLOAD
from django import forms

class FileUploadForm(forms.ModelForm):
    class Meta:
        model = DocumentUpload
        fields = ['title', 'file', 'file_type']
        widgets = {
            'file': forms.FileInput(attrs={'accept': '.pdf,.doc,.docx,.xlsx'}),
        }
    
    def clean_file(self):
        file = self.cleaned_data.get('file')
        
        if file:
            # Check file size (max 5MB)
            if file.size > 5 * 1024 * 1024:
                raise forms.ValidationError('File size must be less than 5MB')
            
            # Check file extension
            if not file.name.endswith(('.pdf', '.doc', '.docx', '.xlsx')):
                raise forms.ValidationError('File type not allowed')
        
        return file

# VIEW WITH FILE UPLOAD
from django.views.generic import CreateView
from django.shortcuts import render

class FileUploadView(LoginRequiredMixin, CreateView):
    model = DocumentUpload
    form_class = FileUploadForm
    template_name = 'upload.html'
    success_url = reverse_lazy('document-list')
    
    def form_valid(self, form):
        form.instance.user = self.request.user
        return super().form_valid(form)

# Function-based view
def upload_file(request):
    if request.method == 'POST':
        form = FileUploadForm(request.POST, request.FILES)
        if form.is_valid():
            document = form.save(commit=False)
            document.user = request.user
            
            # Process file
            file = request.FILES['file']
            print(f"File name: {file.name}")
            print(f"File size: {file.size}")
            
            document.save()
            return redirect('document-list')
    else:
        form = FileUploadForm()
    
    return render(request, 'upload.html', {'form': form})

# SERVING FILES
from django.http import FileResponse

def download_file(request, pk):
    document = get_object_or_404(DocumentUpload, pk=pk)
    
    # Check permission
    if document.user != request.user:
        return HttpResponseForbidden()
    
    # Serve file
    return FileResponse(
        document.file.open('rb'),
        as_attachment=True,
        filename=document.file.name
    )

# SETTINGS FOR FILE UPLOADS
# settings.py
MEDIA_URL = '/media/'
MEDIA_ROOT = os.path.join(BASE_DIR, 'media')

# URL configuration
# urls.py
from django.conf import settings
from django.conf.urls.static import static

urlpatterns = [
    # ... your urls ...
]

if settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL, document_root=settings.MEDIA_ROOT)

# MULTIPLE FILE UPLOAD
class MultipleFileUploadForm(forms.Form):
    files = forms.FileField(widget=forms.ClearableFileInput(attrs={'multiple': True}))

def upload_multiple(request):
    if request.method == 'POST':
        files = request.FILES.getlist('files')
        
        for file in files:
            DocumentUpload.objects.create(
                user=request.user,
                title=file.name,
                file=file
            )
        
        return redirect('document-list')
    
    return render(request, 'upload_multiple.html')

# CUSTOM STORAGE
from django.core.files.storage import FileSystemStorage

def upload_with_custom_path(request):
    if request.method == 'POST':
        file = request.FILES['file']
        
        # Custom storage
        fs = FileSystemStorage(location=os.path.join(BASE_DIR, 'custom_uploads'))
        filename = fs.save(file.name, file)
        file_url = fs.url(filename)
        
        return render(request, 'success.html', {'file_url': file_url})
    
    return render(request, 'upload.html')

# TEMPLATE
"""
<form method="post" enctype="multipart/form-data">
    {% csrf_token %}
    {{ form.as_p }}
    <button type="submit">Upload</button>
</form>

{% if document.file %}
    <a href="{{ document.file.url }}">Download</a>
    <img src="{{ document.file.url }}" alt="Image">
{% endif %}
"""
```

**Key Points:**
- Use `FileField` for any file, `ImageField` for images
- `enctype="multipart/form-data"` required in form
- Validate file size and type in clean method
- Store files in MEDIA directory
- Use context variables for accessing files in templates

---

## Q7: What are Django Signals? How do they work with example?

**Answer:**

Django **Signals** allow decoupled applications to interact by notifying each other when certain actions occur.

```python
from django.db.models.signals import pre_save, post_save, pre_delete, post_delete
from django.dispatch import receiver
from django.core.mail import send_mail
from .models import User, Book

# 1. BASIC SIGNAL - Pre-save
@receiver(pre_save, sender=Book)
def before_book_save(sender, instance, **kwargs):
    print(f"About to save book: {instance.title}")
    # Modify instance before save
    instance.title = instance.title.upper()

# 2. BASIC SIGNAL - Post-save
@receiver(post_save, sender=User)
def create_user_profile(sender, instance, created, **kwargs):
    if created:
        # Create user profile when user is created
        Profile.objects.create(user=instance)
        # Send welcome email
        send_mail(
            'Welcome!',
            f'Welcome {instance.username}',
            'noreply@example.com',
            [instance.email],
        )

# 3. SIGNAL WITH INSTANCE TRACKING
@receiver(post_save, sender=Book)
def update_book_statistics(sender, instance, created, **kwargs):
    if created:
        # Update statistics when new book is created
        Author = sender._meta.get_related_model()
        instance.author.total_books += 1
        instance.author.save()

# 4. PRE-DELETE SIGNAL
@receiver(pre_delete, sender=Book)
def archive_deleted_book(sender, instance, **kwargs):
    # Archive book before deletion
    BookArchive.objects.create(
        title=instance.title,
        author=instance.author,
        archived_date=timezone.now()
    )
    print(f"Archived book: {instance.title}")

# 5. POST-DELETE SIGNAL
@receiver(post_delete, sender=Book)
def update_statistics_on_delete(sender, instance, **kwargs):
    instance.author.total_books -= 1
    instance.author.save()

# 6. SIGNAL WITH CUSTOM DISPATCH_UID
@receiver(post_save, sender=Book, dispatch_uid='book_post_save_signal')
def book_post_save(sender, instance, created, **kwargs):
    if created:
        print(f"New book created: {instance.title}")

# 7. REGISTERING SIGNALS - In apps.py
from django.apps import AppConfig
from django.db.models.signals import post_save

class MyAppConfig(AppConfig):
    default_auto_field = 'django.db.models.BigAutoField'
    name = 'myapp'
    
    def ready(self):
        # Import signals when app is ready
        from . import signals

# 8. SIGNALS FILE - myapp/signals.py
from django.db.models.signals import post_save
from django.dispatch import receiver
from .models import User, UserProfile

@receiver(post_save, sender=User)
def create_profile(sender, instance, created, **kwargs):
    if created:
        UserProfile.objects.create(user=instance)

@receiver(post_save, sender=User)
def save_profile(sender, instance, **kwargs):
    instance.userprofile.save()

# 9. CUSTOM SIGNALS
from django.dispatch import Signal

# Define custom signal
user_activated = Signal()

# Send signal
def activate_user(user):
    user.is_active = True
    user.save()
    
    # Send custom signal
    user_activated.send(sender=User, user=user)

# Listen to custom signal
@receiver(user_activated, sender=User)
def send_activation_email(sender, user, **kwargs):
    send_mail(
        'Account Activated',
        f'Your account {user.username} has been activated',
        'noreply@example.com',
        [user.email],
    )

# 10. COMPLEX SIGNAL EXAMPLE
class BlogPost(models.Model):
    title = models.CharField(max_length=200)
    content = models.TextField()
    author = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    published_at = models.DateTimeField(null=True, blank=True)

@receiver(post_save, sender=BlogPost)
def handle_blog_post_save(sender, instance, created, **kwargs):
    if created:
        # Notify author
        send_mail(
            f'New post: {instance.title}',
            f'Your post "{instance.title}" has been created',
            'noreply@example.com',
            [instance.author.email],
        )
        
        # Log activity
        Activity.objects.create(
            user=instance.author,
            action=f'Created post: {instance.title}'
        )
    else:
        # Check if published_at changed
        original = BlogPost.objects.get(pk=instance.pk)
        if original.published_at != instance.published_at:
            # Post was published
            notify_followers(instance)

# 11. DISCONNECTING SIGNALS (if needed)
from django.db.models.signals import post_save
from .models import User
from .signals import create_profile

post_save.disconnect(create_profile, sender=User)

# 12. SIGNAL TESTING
from django.test import TestCase
from django.db.models.signals import post_save
from .models import User

class SignalTestCase(TestCase):
    def test_create_profile_on_user_save(self):
        user = User.objects.create_user(username='test', password='test')
        # Check if profile was created
        self.assertTrue(hasattr(user, 'profile'))
    
    def test_email_sent_on_user_creation(self):
        with self.assertLogs(level='INFO') as logs:
            user = User.objects.create_user(username='test', email='test@example.com')
        # Verify email was sent
        self.assertIn('Welcome email sent', logs.output[0])
```

**Key Points:**
- Signals enable loose coupling
- Common signals: `pre_save`, `post_save`, `pre_delete`, `post_delete`
- Register in `apps.py` `ready()` method
- Use `dispatch_uid` to prevent duplicate signals
- Can send custom signals
- Avoid complex business logic in signals

---

## Q8: Explain Django's caching framework.

**Answer:**

```python
# CACHING CONFIGURATION - settings.py

# 1. DUMMY CACHE (for development)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.dummy.DummyCache',
    }
}

# 2. MEMORY CACHE (in-process)
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.locmem.LocMemCache',
        'LOCATION': 'unique-snowflake',
    }
}

# 3. REDIS CACHE (production-ready)
CACHES = {
    'default': {
        'BACKEND': 'django_redis.cache.RedisCache',
        'LOCATION': 'redis://127.0.0.1:6379/1',
        'OPTIONS': {
            'CLIENT_CLASS': 'django_redis.client.DefaultClient',
            'SOCKET_CONNECT_TIMEOUT': 5,
            'SOCKET_TIMEOUT': 5,
            'COMPRESSOR': 'django_redis.compressors.zlib.ZlibCompressor',
        }
    }
}

# MEMCACHED
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.memcached.PyMemcacheCache',
        'LOCATION': '127.0.0.1:11211',
    }
}

# DATABASE CACHING
CACHES = {
    'default': {
        'BACKEND': 'django.core.cache.backends.db.DatabaseCache',
        'LOCATION': 'my_cache_table',
    }
}
# Run: python manage.py createcachetable

# CACHE USAGE IN VIEWS

from django.views.decorators.cache import cache_page
from django.shortcuts import render
from django.core.cache import cache

# 1. CACHE PAGE VIEW (entire page)
@cache_page(60 * 5)  # Cache for 5 minutes
def expensive_view(request):
    data = compute_expensive_data()
    return render(request, 'template.html', {'data': data})

# 2. PARTIAL CACHING IN VIEW
def book_list(request):
    # Check cache first
    books = cache.get('all_books')
    
    if books is None:
        # Not in cache, fetch from database
        books = Book.objects.all()
        # Store in cache for 1 hour
        cache.set('all_books', books, 60 * 60)
    
    return render(request, 'books.html', {'books': books})

# 3. CACHE WITH TIMEOUT
def get_user_profile(request):
    cache_key = f'user_{request.user.id}_profile'
    
    profile = cache.get(cache_key)
    if profile is None:
        profile = User.objects.select_related('profile').get(id=request.user.id)
        cache.set(cache_key, profile, 30 * 60)  # 30 minutes
    
    return render(request, 'profile.html', {'profile': profile})

# 4. CACHE WITH INVALIDATION
def update_book(request, pk):
    book = Book.objects.get(pk=pk)
    
    if request.method == 'POST':
        form = BookForm(request.POST, request.FILES, instance=book)
        if form.is_valid():
            form.save()
            
            # Invalidate cache
            cache.delete(f'book_{pk}')
            cache.delete('all_books')
            
            return redirect('book-detail', pk=pk)
    else:
        form = BookForm(instance=book)
    
    return render(request, 'book_form.html', {'form': form})

# 5. CACHE DELETION
cache.delete('key')  # Delete single key
cache.delete_many(['key1', 'key2'])  # Delete multiple
cache.clear()  # Clear all cache

# 6. GET OR SET PATTERN
def get_cached_data(request):
    data = cache.get_or_set(
        'expensive_data',
        compute_expensive_data,
        60 * 60  # Timeout
    )
    return render(request, 'data.html', {'data': data})

# 7. TEMPLATE CACHING
"""
{% load cache %}

{% cache 500 sidebar %}
    <div class="sidebar">
        {% include "sidebar.html" %}
    </div>
{% endcache %}

{% cache 300 product_list product.id %}
    <div class="products">
        {% for product in products %}
            <div>{{ product.name }}</div>
        {% endfor %}
    </div>
{% endcache %}
"""

# 8. CLASS-BASED VIEW CACHING
from django.views.decorators.cache import cache_page
from django.views.generic import ListView
from django.utils.decorators import method_decorator

@method_decorator(cache_page(60 * 5), name='dispatch')
class BookListView(ListView):
    model = Book
    template_name = 'books.html'
    paginate_by = 10

# 9. CACHE KEY FUNCTION
def make_cache_key(request):
    return f"page_{request.user.id}_{request.path}"

# Use with cache_page
@cache_page(60 * 5, key_prefix='custom')
def cached_view(request):
    pass

# 10. LOW-LEVEL CACHE API
from django.core.cache import caches

# Using named cache
cache = caches['default']
cache.set('key', 'value', 300)
value = cache.get('key')

# 11. CACHE STATISTICS
from django_redis import get_redis_connection

redis_conn = get_redis_connection('default')
cache_info = redis_conn.info('stats')

# 12. CACHING WITH QUERYSET
from django.core.cache import cache
from django.views import View

class OptimizedListView(View):
    def get(self, request):
        cache_key = 'books_list_v1'
        books = cache.get(cache_key)
        
        if not books:
            books = list(Book.objects.select_related('author').values())
            cache.set(cache_key, books, 60 * 60)
        
        return render(request, 'books.html', {'books': books})

# 13. CACHE MIDDLEWARE (cache entire page)
MIDDLEWARE = [
    # ... other middleware ...
    'django.middleware.cache.UpdateCacheMiddleware',
    # ... other middleware ...
    'django.middleware.cache.FetchFromCacheMiddleware',
]

CACHE_MIDDLEWARE_ALIAS = 'default'
CACHE_MIDDLEWARE_SECONDS = 600
CACHE_MIDDLEWARE_KEY_PREFIX = 'mysite'
```

**Key Points:**
- Choose Redis for production
- Cache pages, querysets, expensive computations
- Invalidate cache when data changes
- Use cache decorators for views
- Monitor cache hit/miss rates
- Set appropriate timeout values

---

# FLASK

---

## Q1: What is the Application Factory Pattern in Flask and why is it important?

**Answer:**

The Application Factory Pattern is a way to create Flask app instances using a function, rather than a global object. This enables:
- Multiple app instances (testing, production)
- Easier configuration
- Extension initialization
- Avoids circular imports

```python
# app/__init__.py
from flask import Flask

def create_app(config_name=None):
    app = Flask(__name__)
    app.config.from_object(config_name or 'config.DevelopmentConfig')
    # Initialize extensions
    # from .extensions import db, migrate
    # db.init_app(app)
    # migrate.init_app(app, db)
    # Register blueprints
    # from .routes import main_bp
    # app.register_blueprint(main_bp)
    return app

# run.py
from app import create_app
app = create_app()

if __name__ == '__main__':
    app.run()
```

**Key Points:**
- Use for scalable, testable Flask projects
- Allows different configs for dev/test/prod
- Required for many extensions (Flask-Migrate, Flask-Login)

---

## Q2: How does routing and Blueprints work in Flask?

**Answer:**

Routing maps URLs to Python functions. Blueprints organize routes into reusable modules.

```python
# app/routes.py
from flask import Blueprint, render_template, request

main_bp = Blueprint('main', __name__)

@main_bp.route('/')
def home():
    return render_template('home.html')

@main_bp.route('/user/<username>')
def user_profile(username):
    return f"Profile: {username}"

@main_bp.route('/search')
def search():
    query = request.args.get('q')
    return f"Search results for: {query}"

# Register blueprint in factory
# app.register_blueprint(main_bp)
```

**Key Points:**
- Blueprints enable modular apps (auth, admin, api)
- Use `url_for('main.home')` for reverse routing
- Blueprints can have their own templates/static

---

## Q3: How do you use Jinja2 templates in Flask?

**Answer:**

Jinja2 is Flask's template engine for rendering HTML with dynamic data.

```python
# app/templates/home.html
"""
<!DOCTYPE html>
<html>
<head><title>Home</title></head>
<body>
    <h1>Welcome, {{ user.username }}!</h1>
    {% if user.is_admin %}
        <a href="/admin">Admin Panel</a>
    {% endif %}
    <ul>
        {% for item in items %}
            <li>{{ item }}</li>
        {% endfor %}
    </ul>
</body>
</html>
"""

# app/routes.py
from flask import render_template

@app.route('/')
def home():
    user = {'username': 'John', 'is_admin': True}
    items = ['A', 'B', 'C']
    return render_template('home.html', user=user, items=items)
```

**Key Points:**
- Use `{{ variable }}` for data, `{% ... %}` for logic
- Supports filters, inheritance, macros
- Prevents XSS by auto-escaping

---

## Q4: How does Flask handle requests and responses?

**Answer:**

Flask provides `request` and `response` objects for HTTP data.

```python
from flask import request, Response, jsonify, make_response

@app.route('/submit', methods=['POST'])
def submit():
    data = request.form['data']
    json_data = request.get_json()
    headers = request.headers
    cookies = request.cookies
    # Custom response
    resp = make_response(jsonify({'status': 'ok'}), 200)
    resp.set_cookie('user_id', '123')
    return resp
```

**Key Points:**
- Use `request.args`, `request.form`, `request.json` for data
- Use `make_response`, `jsonify` for custom responses
- Set cookies/headers as needed

---

## Q5: How do you manage sessions and cookies in Flask?

**Answer:**

Sessions store user data between requests. Cookies are key-value pairs sent to the browser.

```python
from flask import session, redirect, url_for

@app.route('/login', methods=['POST'])
def login():
    user_id = request.form['user_id']
    session['user_id'] = user_id
    return redirect(url_for('profile'))

@app.route('/logout')
def logout():
    session.pop('user_id', None)
    return redirect(url_for('home'))

# Set a cookie
resp = make_response('Set cookie')
resp.set_cookie('theme', 'dark', max_age=3600)

# Read a cookie
theme = request.cookies.get('theme')
```

**Key Points:**
- Flask uses secure cookies for sessions
- Set `SECRET_KEY` in config for session security
- Use `session` for server-side data

---

## Q6: What are Flask extensions? Give examples of SQLAlchemy and WTForms usage.

**Answer:**

Flask extensions add features like ORM, forms, authentication, etc.

### Flask-SQLAlchemy (ORM)
```python
from flask_sqlalchemy import SQLAlchemy

db = SQLAlchemy()

class User(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    username = db.Column(db.String(80), unique=True, nullable=False)
    email = db.Column(db.String(120), unique=True, nullable=False)

# In app factory
app.config['SQLALCHEMY_DATABASE_URI'] = 'sqlite:///app.db'
db.init_app(app)

# CRUD
user = User(username='john', email='john@example.com')
db.session.add(user)
db.session.commit()
users = User.query.all()
```

### WTForms (Form handling)
```python
from flask_wtf import FlaskForm
from wtforms import StringField, PasswordField, SubmitField
from wtforms.validators import DataRequired, Email

class LoginForm(FlaskForm):
    email = StringField('Email', validators=[DataRequired(), Email()])
    password = PasswordField('Password', validators=[DataRequired()])
    submit = SubmitField('Login')

# In route
@app.route('/login', methods=['GET', 'POST'])
def login():
    form = LoginForm()
    if form.validate_on_submit():
        # Authenticate user
        pass
    return render_template('login.html', form=form)
```

**Key Points:**
- Extensions must be initialized with app
- Popular: Flask-Migrate, Flask-Login, Flask-Mail, Flask-Caching

---

## Q7: How does middleware work in FastAPI?

**Answer:**

Middleware in FastAPI allows you to process requests and responses globally (logging, CORS, authentication, etc.).

```python
from fastapi import FastAPI, Request
from starlette.middleware.cors import CORSMiddleware

app = FastAPI()

@app.middleware('http')
async def log_requests(request: Request, call_next):
    print(f'Request: {request.method} {request.url}')
    response = await call_next(request)
    print(f'Response status: {response.status_code}')
    return response

# Built-in CORS middleware
app.add_middleware(
    CORSMiddleware,
    allow_origins=['*'],
    allow_credentials=True,
    allow_methods=['*'],
    allow_headers=['*'],
)
```

**Key Points:**
- Use `@app.middleware('http')` for custom middleware
- Use built-in middleware for CORS, GZip, etc.
- Middleware runs before and after request processing

---

## Q8: How do you test FastAPI applications?

**Answer:**

Use FastAPI's TestClient (based on Starlette) and pytest/unittest.

```python
from fastapi.testclient import TestClient
from app import app

client = TestClient(app)

def test_hello():
    response = client.get('/hello')
    assert response.status_code == 200
    assert response.json() == {'message': 'Hello'}

# Testing POST

def test_create_item():
    response = client.post('/items/', json={'name': 'Book', 'price': 10.0, 'is_offer': False})
    assert response.status_code == 200
    assert response.json()['name'] == 'Book'
```

**Key Points:**
- Use TestClient for requests
- Use pytest for fixtures and assertions
- Test endpoints, models, error handling

---
