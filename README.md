# alx_travel_app_0x01
##  build API views to manage listings and bookings with Swagger documentation
Step 1: Duplicate the Project
First, duplicate your project:
cp -r alx_travel_app_0x00 alx_travel_app_0x01
cd alx_travel_app_0x01

Step 2: Install Required Dependencies
Add to your requirements.txt:
Django==4.2.0
djangorestframework==3.14.0
drf-yasg==1.21.11

Install dependencies:
pip install -r requirements.txt

Step 3: Create ViewSets in listings/views.py
from rest_framework import viewsets, permissions
from .models import Listing, Booking
from .serializers import ListingSerializer, BookingSerializer

class ListingViewSet(viewsets.ModelViewSet):
    """
    API endpoint for managing property listings.
    
    Provides CRUD operations for Listing model.
    """
    queryset = Listing.objects.all()
    serializer_class = ListingSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

class BookingViewSet(viewsets.ModelViewSet):
    """
    API endpoint for managing property bookings.
    
    Provides CRUD operations for Booking model.
    """
    queryset = Booking.objects.all()
    serializer_class = BookingSerializer
    permission_classes = [permissions.IsAuthenticatedOrReadOnly]

    def perform_create(self, serializer):
        """Set the user who created the booking."""
        serializer.save(user=self.request.user)

Step 4: Create Serializers in listings/serializers.py
from rest_framework import serializers
from .models import Listing, Booking

class ListingSerializer(serializers.ModelSerializer):
    class Meta:
        model = Listing
        fields = '__all__'
        read_only_fields = ('owner', 'created_at', 'updated_at')

class BookingSerializer(serializers.ModelSerializer):
    listing_title = serializers.CharField(source='listing.title', read_only=True)
    user_email = serializers.CharField(source='user.email', read_only=True)
    
    class Meta:
        model = Booking
        fields = '__all__'
        read_only_fields = ('user', 'created_at', 'updated_at')

Step 5: Configure URLs in alx_travel_app/urls.py
from django.contrib import admin
from django.urls import path, include
from rest_framework import routers
from drf_yasg.views import get_schema_view
from drf_yasg import openapi
from rest_framework import permissions

# Import views
from listings.views import ListingViewSet, BookingViewSet

# Router configuration
router = routers.DefaultRouter()
router.register(r'listings', ListingViewSet)
router.register(r'bookings', BookingViewSet)

# Swagger configuration
schema_view = get_schema_view(
    openapi.Info(
        title="ALX Travel App API",
        default_version='v1',
        description="API documentation for ALX Travel App",
        terms_of_service="https://www.google.com/policies/terms/",
        contact=openapi.Contact(email="contact@alxtravel.local"),
        license=openapi.License(name="BSD License"),
    ),
    public=True,
    permission_classes=(permissions.AllowAny,),
)

urlpatterns = [
    path('admin/', admin.site.urls),
    path('api/', include(router.urls)),
    path('api/auth/', include('rest_framework.urls')),  # For login/logout
    
    # Swagger URLs
    path('swagger.json', schema_view.without_ui(cache_timeout=0), name='schema-json'),
    path('swagger/', schema_view.with_ui('swagger', cache_timeout=0), name='schema-swagger-ui'),
    path('redoc/', schema_view.with_ui('redoc', cache_timeout=0), name='schema-redoc'),
]

Step 6: Update Settings in alx_travel_app/settings.py
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    
    # Third party apps
    'rest_framework',
    'drf_yasg',
    
    # Local apps
    'listings',
]

# REST Framework configuration
REST_FRAMEWORK = {
    'DEFAULT_PERMISSION_CLASSES': [
        'rest_framework.permissions.IsAuthenticatedOrReadOnly',
    ],
    'DEFAULT_AUTHENTICATION_CLASSES': [
        'rest_framework.authentication.SessionAuthentication',
        'rest_framework.authentication.BasicAuthentication',
    ],
    'DEFAULT_PAGINATION_CLASS': 'rest_framework.pagination.PageNumberPagination',
    'PAGE_SIZE': 10
}

# Swagger settings
SWAGGER_SETTINGS = {
    'SECURITY_DEFINITIONS': {
        'Basic': {
            'type': 'basic'
        }
    },
    'USE_SESSION_AUTH': True,
}

Step 7: Update Models (if needed) in listings/models.py
Make sure your models have proper string representations for the API:
from django.db import models
from django.contrib.auth.models import User

class Listing(models.Model):
    title = models.CharField(max_length=200)
    description = models.TextField()
    price = models.DecimalField(max_digits=10, decimal_places=2)
    location = models.CharField(max_length=200)
    owner = models.ForeignKey(User, on_delete=models.CASCADE)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return self.title

class Booking(models.Model):
    listing = models.ForeignKey(Listing, on_delete=models.CASCADE)
    user = models.ForeignKey(User, on_delete=models.CASCADE)
    start_date = models.DateField()
    end_date = models.DateField()
    total_price = models.DecimalField(max_digits=10, decimal_places=2)
    created_at = models.DateTimeField(auto_now_add=True)
    updated_at = models.DateTimeField(auto_now=True)

    def __str__(self):
        return f"{self.user.username} - {self.listing.title}"

Step 8: Run Migrations and Test
# Run migrations
python manage.py makemigrations
python manage.py migrate

# Create superuser for testing
python manage.py createsuperuser

# Run server
python manage.py runserver

Step 9: Test Endpoints
Available API Endpoints:
GET/POST /api/listings/ - List/Create listings

GET/PUT/DELETE /api/listings/{id}/ - Retrieve/Update/Delete listing

GET/POST /api/bookings/ - List/Create bookings

GET/PUT/DELETE /api/bookings/{id}/ - Retrieve/Update/Delete booking

GET /swagger/ - Swagger UI documentation

GET /redoc/ - ReDoc documentation

Testing with Postman:
Get all listings:
GET http://localhost:8000/api/listings/
Create a listing:
POST http://localhost:8000/api/listings/
Headers: Content-Type: application/json
Body: {
  "title": "Beautiful Beach House",
  "description": "Amazing beachfront property",
  "price": "150.00",
  "location": "Mombasa"
}
Create a booking:
POST http://localhost:8000/api/bookings/
Headers: Content-Type: application/json
Body: {
  "listing": 1,
  "start_date": "2024-01-15",
  "end_date": "2024-01-20",
  "total_price": "750.00"
}

Step 10: View Documentation
Visit http://localhost:8000/swagger/ to see the interactive Swagger documentation where you can test all endpoints directly from the browser.

This setup provides:

✅ Complete CRUD operations for both models

✅ RESTful API endpoints under /api/

✅ Automatic Swagger documentation

✅ Proper authentication and permissions

✅ Easy testing with Postman or Swagger UI
