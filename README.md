# CVE-2024-22513

The version of [djangorestframework-simplejwt](https://github.com/jazzband/djangorestframework-simplejwt) up to 5.3.1 is vulnerable. This vulnerability has the potential to cause various security issues, including Business Object Level Authorization (BOLA), Business Function Level Authorization (BFLA), Information Disclosure, etc. The vulnerability arises from the fact that a user can access web application resources even after their account has been disabled, primarily due to the absence of proper user validation checks.

## Insecure Design

If a programmer generates a JWT token for an inactive user using `AccessToken` class and `for_user` method then a JWT token is returned which can be used for authentication across the django and django rest framework application.

-   Start Django shell

```bash
python manage.py shell
```

-   Create inactive user and generate token for the user

```python
from django.contrib.auth.models import User
from rest_framework_simplejwt.tokens import AccessToken

# create inactive user
inactive_user_id = User.objects.create_user('testuser', 'test@example.com', 'testPassw0rd!', is_active=False).id

# django application programmer generates token for the inactive user
AccessToken.for_user(User.objects.get(id=inactive_user_id))  # error should be raised since user is inactive

# django application verifying user token
AccessToken.for_user(User.objects.get(id=inactive_user_id)).verify() # no exception is raised during verification of inactive user token
```

## Impact

Impact would vary from application to application but general impacts are listed below:

-   Bypass of Account Deactivation:
    The vulnerability allows an attacker to generate a JWT token for an inactive user. This means that even if an account has been deactivated (e.g., due to suspicious activity or user request), an attacker can still obtain a valid token for that user.

-   Authentication Bypass:
    Since the token is successfully generated for the inactive user, an attacker can use this token to authenticate requests to the Django and Django Rest Framework applications. This leads to an authentication bypass, allowing unauthorized access to protected resources.

-   Authorization Issues:
    Depending on how your application is structured, an attacker with a token for an inactive user might have unintended access to certain functionalities or data. This can result in Business Object Level Authorization (BOLA) issues, where an attacker gains access to resources they shouldn't have.

-   Information Disclosure:
    The ability to authenticate as an inactive user could lead to information disclosure. For example, if there are certain resources or endpoints that should only be accessible to active users, an attacker could gain access to sensitive information.

-   Security Misconfigurations:
    This vulnerability indicates a potential misconfiguration in the design of the authentication and authorization mechanisms. It could signal a broader issue in the security architecture of the application.
