# Create project directory
mkdir library_project && cd library_project

# Set up virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Django and DRF
pip install django djangorestframework


django-admin startproject main
python manage.py startapp books


Gemini said
Building a Library Management system with Django and Django REST Framework (DRF) is a classic way to learn how the backend connects to the frontend. Since you want your own UI, we will build the API and then use Django's template system with a bit of JavaScript (or standard HTML forms) to interact with it.

Here is your step-by-step roadmap.

1. Environment Setup
First, let's get your project folder ready and install the necessary packages.

Bash
# Create project directory
mkdir library_project && cd library_project

# Set up virtual environment
python -m venv venv
source venv/bin/activate  # On Windows: venv\Scripts\activate

# Install Django and DRF
pip install django djangorestframework
2. Project Initialization
We’ll create the project and a specific app called books.

Add 'rest_framework' and 'books' to your INSTALLED_APPS in core/settings.py.


In books/models.py, define what a "Book" looks like.

Python
from django.db import models

class Book(models.Model):
    title = models.CharField(max_length=200)
    author = models.CharField(max_length=100)
    description = models.TextField()
    published_date = models.DateField()

    def __str__(self):
        return self.title
Run python manage.py makemigrations and python manage.py migrate after saving this.

4. The API Layer (Serializers & Views)
The Serializer converts your database objects into JSON so the UI can read them.

Create books/serializers.py:

Python
from rest_framework import serializers
from .models import Book

class BookSerializer(serializers.ModelSerializer):
    class Meta:
        model = Book
        fields = '__all__'
In books/views.py:
We will use a ModelViewSet, which handles GET, POST, PUT, and DELETE automatically.

Python
from rest_framework import viewsets
from django.shortcuts import render
from .models import Book
from .serializers import BookSerializer

# This handles the API logic
class BookViewSet(viewsets.ModelViewSet):
    queryset = Book.objects.all()
    serializer_class = BookSerializer

# This renders your actual HTML UI
def library_ui(request):
    return render(request, 'books/index.html')
5. Routing (URLs)
We need to tell Django where the API lives and where the UI lives.

In core/urls.py:

Python
from django.contrib import admin
from django.urls import path, include
from rest_framework.routers import DefaultRouter
from books.views import BookViewSet, library_ui

router = DefaultRouter()
router.register(r'api/books', BookViewSet)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('', library_ui), # The UI home page
    path('', include(router.urls)), # The API endpoints
]
6. Creating the UI (HTML + JavaScript)
Create a folder structure: books/templates/books/index.html.

To make this "Modern CRUD," we will use fetch() in JavaScript. This allows you to Add, Delete, and Update without the page refreshing like an old-school website.

index.html highlights:

HTML
<!DOCTYPE html>
<html>
<head>
    <title>Library Manager</title>
    <style>
        body { font-family: sans-serif; margin: 40px; }
        .book-card { border: 1px solid #ccc; padding: 10px; margin: 10px 0; }
    </style>
</head>
<body>
    <h1>Library CRUD UI</h1>
    
    <div id="book-form">
        <input type="text" id="title" placeholder="Title">
        <input type="text" id="author" placeholder="Author">
        <textarea id="description" placeholder="Description"></textarea>
        <input type="date" id="published_date">
        <button onclick="addBook()">Add Book</button>
    </div>

    <hr>
    <div id="book-list"></div>

    <script>
        const apiURL = '/api/books/';

        // GET: Fetch and display books
        async function fetchBooks() {
            const response = await fetch(apiURL);
            const books = await response.json();
            const list = document.getElementById('book-list');
            list.innerHTML = '';
            books.forEach(book => {
                list.innerHTML += `
                    <div class="book-card">
                        <h3>${book.title}</h3>
                        <p>${book.author}</p>
                        <button onclick="deleteBook(${book.id})">Delete</button>
                    </div>
                `;
            });
        }

        // POST: Add a new book
        async function addBook() {
            const data = {
                title: document.getElementById('title').value,
                author: document.getElementById('author').value,
                description: document.getElementById('description').value,
                published_date: document.getElementById('published_date').value,
            };
            await fetch(apiURL, {
                method: 'POST',
                headers: {'Content-Type': 'application/json'},
                body: JSON.stringify(data)
            });
            fetchBooks();
        }

        // DELETE: Remove a book
        async function deleteBook(id) {
            await fetch(`${apiURL}${id}/`, { method: 'DELETE' });
            fetchBooks();
        }

        fetchBooks(); // Load books on startup
    </script>
</body>
</html>


Summary of Operations
Operation	HTTP Method	Endpoint
Get All	GET	/api/books/
Get One	GET	/api/books/1/
Create	POST	/api/books/
Update	PUT	/api/books/1/
Delete	DELETE	/api/books/1/


A Quick Tip
Since you are building a UI to talk to an API, make sure to handle CSRF tokens if you decide to go beyond a local project. For local testing, DRF usually allows these requests if they are coming from the same origin.
ing ### API with UI (3).txt…]()
[### API with UI (3).txt](https://github.com/user-attachments/files/25456523/API.with.UI.3.txt)
[### API with UI (3).txt](https://github.com/user-attachments/files/25456523/API.with.UI.3.txt)
