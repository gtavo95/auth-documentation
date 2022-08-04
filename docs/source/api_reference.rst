API Reference
=============

.. autofunction:: auth.views.Register
.. autofunction:: auth.views.Login
.. autofunction:: auth.views.Logout
.. autofunction:: auth.views.AuthTest

**Example Response**

.. sourcecode:: http

    HTTP/1.1 200 OK      
    Content-Type: text/javascript

    {
        "access_token": "lXwmcGFxUuUaOl14KSRDPCyEI6RyI7",
        "expires_in": 36000,
        "token_type": "Bearer",
        "scope": "openid read write introspection"
    }