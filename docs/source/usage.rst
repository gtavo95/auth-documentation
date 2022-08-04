Usage
=====

First thing first, you need to define which authorization flow is appropiate for you application: ``Authorization code flow`` or ``client-credential flow``

The ``Authorization code flow`` is best suited for web applications and mobile apps. Use it when you need access to the user information. The flow start when the client 
try to enter to a restricted resource (hosted in the workinglive service provider API), followed by that the user is redirected to a login page, where gets authenticated and asked for permissions.
If the user gives consent, a grant code is generated (by the authorization server) and send it to the client. With the grant code the client can request to the authorization server an access token (for authorization) and a tokenid (for authentication).
With the tokens generated the client can access to restricted resources.

The ``Client-Credential flow`` is preferable in machine to machine comunication. The flow start when the client request a token to the authorization server, the authorization server generate the token 
and send it to the client, with the token on hand the client can access to restricted resources hosted in the workinglive service provider API. There is no tokenid.


Get your Credentials
--------------------

No matter what flow you will work on, first you need Aplication credentials: 

.. code-block:: python

    example_id="qJt8PYJaBDMKBgIgGwDeuNo9Fm8xjU0f97ycxJlZ"
    example_secret="DxFW4ZPeVOrhDHYLuuc3LePK7C2ty5NI4HU9iePMKbxVhQerJS2hbCoaEyCDF6L2E09HSM8FxVt7j7Cd7CUeoiDcc6yyNxnFdnTPJqToTruH9wqUW06C74cfcvgNWLIZ"


To get your own credential send an email to: mateo@workinglive.com, remember to save your credentials in secret.


.. _configuration:

Client-credentials Configuration
--------------------------------

Encode Credentials 
^^^^^^^^^^^^^^^^^^

In order to generate tokens using the client-credential flow you need to encode your credentials using base64 as followed:

.. tab:: Python
    
    .. code-block:: python   

        example_id = "jjynVYkBQlGl8uvtAeul8QyfRY3iiQeNq8h6C5hP"
        example_secret = "EvZoQCOM6BcBSbE7IARYfnztueiXdTr531zFpnRpHwhGo9lJ7lok5uXXR3xTKHhmxgLyO1ti89Q0CUFVNLaUlPE5quqwQX4t9cJvGrz9tNvLrXoB3JR4qAOlolezDnTP"
        credential = "{0}:{1}".format(example_id, example_secret)
        credential = base64.b64encode(credential.encode("utf-8"))

.. tab:: NodeJs
    
    .. code-block:: javascript   

        exampleId = 'jjynVYkBQlGl8uvtAeul8QyfRY3iiQeNq8h6C5hP'
        exampleSecret = 'EvZoQCOM6BcBSbE7IARYfnztueiXdTr531zFpnRpHwhGo9lJ7lok5uXXR3xTKHhmxgLyO1ti89Q0CUFVNLaUlPE5quqwQX4t9cJvGrz9tNvLrXoB3JR4qAOlolezDnTP'
        credential = `${exampleId}:${exampleSecret}`
        credential = Buffer.from(credential).toString('base64')
        

Request Access Token
^^^^^^^^^^^^^^^^^^^^

Then with the credential variable created you can request an access token to the authentication server. 

.. sourcecode:: http

      GET /o/token/ HTTP/1.1
      Host: http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000
      Accept: application/json
      Authorization: Basic {{ credential }}

**Example request**

.. tab:: Curl

    .. code-block:: bash

        curl --location --request POST 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/' \
        --header 'Authorization: Basic {{ credential }}' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "grant_type":"client_credentials"
        }'

.. tab:: Python

    .. code-block:: python

        import requests
        import json

        url = "http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/"
        
        payload = json.dumps({
            "grant_type": "client_credentials"
        })

        headers = {
            'Authorization': 'Basic {0}'.format(credential),
            'Content-Type': 'application/json'
        }
        
        response = requests.request("POST", url, headers=headers, data=payload)
        
        print(response.text)

.. tab:: NodeJs

    .. code-block:: javascript
        
        var axios = require('axios');

        var data = JSON.stringify({
            "grant_type": "client_credentials"
        });

        var config = {
            method: 'post',
            url: 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/',
            headers: { 
                'Authorization': `Basic ${credential}`, 
                'Content-Type': 'application/json'
            },
            data : data
        };

        axios(config)
        .then(function (response) {
            console.log(JSON.stringify(response.data));
        })
        .catch(function (error) {
            console.log(error);
        });

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


Authorization Code Configuration
--------------------------------

If you plan to use the authorization code flow, you need to tell mateo@workinglive.com to set a return_url in the authorization server, 
this page will be rendered after user consent and login. The next thing you need to do is generate the authentication url as shown:

Authentication URL
^^^^^^^^^^^^^^^^^^

.. code-block:: bash

    url = host + "/o/authorize/?response_type=code&code_challenge=" + 
        code_challenge + "&client_id=" + 
        client_id + "&redirect_uri=" + 
        redirect_uri + "&approval_prompt=auto#&prompt=prompt"


You need to redirect the user to this url whenever the user tries to access a protected resource. Let's understand and recreate this url.


Generate Code Verifier
^^^^^^^^^^^^^^^^^^^^^^

First we need to generate the ``code_challenge``. To do so, you must generate a random string between 43 and 128 characters, we will call it ``code_verifier``, save it somewhere since we
are going to use it later. 

.. tab:: Python

    .. code-block:: python

        code_verifier = ''.join(random.choice(string.ascii_uppercase + string.digits) for _ in range(random.randint(43, 128)))
        code_verifier = base64.urlsafe_b64encode(code_verifier.encode('utf-8'))

The second step is to encode the ``code_verifier`` and produce the ``code_challenge``.

Generate Code Challenge
^^^^^^^^^^^^^^^^^^^^^^^

.. tab:: Python

    .. code-block:: python

        code_challenge = hashlib.sha256(code_verifier).digest()
        code_challenge = base64.urlsafe_b64encode(code_challenge).decode('utf-8').replace('=', '')


Third step is set your ``redirect_uri`` and your  ``client_id``.

URL Query parameters (promnt login, promnt grant, etc...)
^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^

Now we have all the query parameters ready! It is important to note that the ``response_type`` parameter is set to ``code`` this tell the authorization server that we are going to start 
an authorization code flow, also notice that the ``approval_prompt`` parameter is set to ``auto``, this tells the authorization server to request the user's consent only once and finally the ``promnt`` parameter
tells the authorization server to display the **prompt** to the user, you can change this to ``login``  to redirect the user to a login page hosted by WorkingLive Authorization Services.

Request Access Token 
^^^^^^^^^^^^^^^^^^^^

If the user gives consent the authorization server will generate and send you a grant code. With the grant code you can request an access token as shown:

**Example request**

.. tab:: Curl

    .. code-block:: bash

        curl --location --request POST 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb", 
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA", 
            "code": "FAdG7g2CA6RUhSwv7Vrxwu7Tmym39r", 
            "redirect_uri": "https://test.com", 
            "grant_type": "authorization_code", 
            "code_verifier": "5cv9M6gB4DWoQ1ziri0qsyUMv1IiJG0CQtpJSj2rYig"
        }'

.. tab:: Python

    .. code-block:: python

        import requests
        import json

        url = "http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/"

        payload = json.dumps({
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb",
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA",
            "code": "FAdG7g2CA6RUhSwv7Vrxwu7Tmym39r",
            "redirect_uri": "https://test.com",
            "grant_type": "authorization_code",
            "code_verifier": "5cv9M6gB4DWoQ1ziri0qsyUMv1IiJG0CQtpJSj2rYig"
        })
        headers = {
            'Content-Type': 'application/json'
        }

        response = requests.request("POST", url, headers=headers, data=payload)

        print(response.text)

.. tab:: NodeJs

    .. code-block:: javascript

        var axios = require('axios');
        var data = JSON.stringify({
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb",
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA",
            "code": "FAdG7g2CA6RUhSwv7Vrxwu7Tmym39r",
            "redirect_uri": "https://test.com",
            "grant_type": "authorization_code",
            "code_verifier": "5cv9M6gB4DWoQ1ziri0qsyUMv1IiJG0CQtpJSj2rYig"
        });

        var config = {
            method: 'post',
            url: 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/token/',
            headers: { 
                'Content-Type': 'application/json'
            },
            data : data
        };

        axios(config)
        .then(function (response) {
            console.log(JSON.stringify(response.data));
        })
        .catch(function (error) {
            console.log(error);
        });


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


Request user information
^^^^^^^^^^^^^^^^^^^^^^^^

If you need to access aditional user information use this endpoint ``/o/userinfo/``` 

**Example Request**

.. tab:: Curl

    .. code-block:: bash

        curl --location --request POST 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/userinfo/' \
        --header 'Authorization: Bearer qcdlDp4R5mCAC8iPrlhTcVGOIdF5Gp' \
        --header 'Content-Type: application/json' \
        --data-raw '{
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb", 
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA", 
            "code": "jAGuWnyqvZI9ROiCfLGo76xgoVv5mN", 
            "redirect_uri": "https://test.com", 
            "grant_type": "authorization_code", 
            "code_verifier": "UuZ390GATtWgRpbiFG6VPPJWZe0vYQox_WlG8TiJc8c"
        }'

.. tab:: Python

    .. code-block:: python

        import requests
        import json

        url = "http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/userinfo/"

        payload = json.dumps({
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb",
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA",
            "code": "jAGuWnyqvZI9ROiCfLGo76xgoVv5mN",
            "redirect_uri": "https://test.com",
            "grant_type": "authorization_code",
            "code_verifier": "UuZ390GATtWgRpbiFG6VPPJWZe0vYQox_WlG8TiJc8c"
        })
        headers = {
            'Authorization': 'Bearer qcdlDp4R5mCAC8iPrlhTcVGOIdF5Gp',
            'Content-Type': 'application/json'
        }

        response = requests.request("POST", url, headers=headers, data=payload)

        print(response.text)


.. tab:: NodeJs

    .. code-block:: javascript

        var axios = require('axios');
        var data = JSON.stringify({
            "client_id": "KC6W7ZDVIw08K4Skt0qEV3sQQGkLO6JZpvuZerzb",
            "client_secret": "yeT6F6WNzM1SZklIlD83O4xhmvRuuYZT9EzI3L77KswVgy7i5HHuBPhVQzIxpJjE3Sfc4jdWl3oFjVX3d5MxHaNm1JEc5hjvBbaPTtgm54QuWc5nHFaZTJHLP3hTvWTA",
            "code": "jAGuWnyqvZI9ROiCfLGo76xgoVv5mN",
            "redirect_uri": "https://test.com",
            "grant_type": "authorization_code",
            "code_verifier": "UuZ390GATtWgRpbiFG6VPPJWZe0vYQox_WlG8TiJc8c"
        });

        var config = {
            method: 'post',
            url: 'http://auth-balancer-c6b1e2cf1a06e31d.elb.ca-central-1.amazonaws.com:8000/o/userinfo/',
            headers: { 
                'Authorization': 'Bearer qcdlDp4R5mCAC8iPrlhTcVGOIdF5Gp', 
                'Content-Type': 'application/json'
            },
                data : data
        };

        axios(config)
        .then(function (response) {
            console.log(JSON.stringify(response.data));
        })
        .catch(function (error) {
            console.log(error);
        });



