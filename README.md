# TechHire — Freemium Job Board API

A production-ready REST API built with Django, DRF, and `rest_framework_simplejwt`.

---

## Project structure

```
techhire/
├── core/
│   ├── settings.py       # All DRF, JWT, pagination settings
│   └── urls.py           # Root URL config (JWT + jobs app)
├── jobs/
│   ├── migrations/
│   │   └── 0001_initial.py
│   ├── management/commands/
│   │   └── seed_jobs.py  # python manage.py seed_jobs
│   ├── admin.py
│   ├── apps.py           # Loads signals on startup
│   ├── models.py         # JobPosting + UserProfile
│   ├── serializers.py    # Field-level masking via to_representation
│   ├── signals.py        # Auto-create UserProfile on User save
│   ├── urls.py
│   └── views.py
├── requirements.txt
└── TechHire_API.postman_collection.json
```

---

## Quick-start

```bash
# 1. Install dependencies
pip install -r requirements.txt

# 2. Apply migrations
python manage.py migrate

# 3. Create a superuser (for the admin panel)
python manage.py createsuperuser

# 4. Seed sample job postings
python manage.py seed_jobs

# 5. Run the development server
python manage.py runserver
```

### Promote a user to Premium (via Django shell)

```python
python manage.py shell
>>> from django.contrib.auth.models import User
>>> user = User.objects.get(username="your_username")
>>> user.profile.membership_type = "Premium"
>>> user.profile.save()
```

---

## API Endpoints

| Method | URL | Auth required | Description |
|--------|-----|--------------|-------------|
| POST | `/api/token/` | No | Obtain JWT access + refresh token |
| POST | `/api/token/refresh/` | No | Rotate access token |
| POST | `/api/token/verify/` | No | Verify token validity |
| GET | `/api/jobs/` | Optional | List job postings (paginated, filtered) |
| GET | `/api/jobs/<id>/` | Optional | Retrieve single job posting |
| GET | `/api/me/` | Yes (JWT) | Get current user's membership info |

---

## Filtering, search, and ordering

All options work on both the list and detail endpoints.

| Query param | Example | Effect |
|-------------|---------|--------|
| `location` | `?location=Remote` | Exact match filter |
| `search` | `?search=python` | Full-text across `title` + `description` |
| `ordering` | `?ordering=-created_at` | Sort (default: newest first) |
| `page` | `?page=2` | Pagination (10 per page) |

---

## Field-level masking logic

The `JobPostingSerializer.to_representation` method applies this decision tree:

```
Request context missing?         → MASK
User not authenticated?          → MASK
User has no UserProfile?         → MASK
UserProfile.membership_type != "Premium"? → MASK
Otherwise                        → REVEAL
```

Masked fields: `company_name`, `salary_range`, `application_link`
Always visible: `id`, `title`, `description`, `location`, `created_at`

Masked value: `"🔒 Premium Feature"`

---

## JWT settings summary

| Setting | Value |
|---------|-------|
| Access token lifetime | 60 minutes |
| Refresh token lifetime | 7 days |
| Rotate refresh tokens | Yes |
| Blacklist after rotation | Yes |
| Algorithm | HS256 |
| Header format | `Authorization: Bearer <token>` |

---

## Postman collection

Import `TechHire_API.postman_collection.json` into Postman.

Set the `base_url` collection variable to `http://127.0.0.1:8000`.

Run **Request 1** first — the test script automatically stores `access_token`
and `refresh_token` as collection variables for use in subsequent requests.
