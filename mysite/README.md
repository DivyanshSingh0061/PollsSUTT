# Django Polls – SUTT Backend Task 1.5

Polls app built by following the official Django tutorial (Parts 1–8).

## How to run

    python -m pip install django django-debug-toolbar
    python manage.py migrate
    python manage.py createsuperuser
    python manage.py runserver

Then open http://127.0.0.1:8000/polls/ and http://127.0.0.1:8000/admin/

## How the web works

The web works by having a browser ask a server for something and the server send back a response.
In my polls app, clicking a question feels like moving from one page to another,
but what actually happens is that Chrome asks Django for that question’s page.
Django gets the question and its choices from the database,
puts them into an HTML page and sends it back. Chrome displays the page, and when I click Vote,
that starts a new request.

### What actually happens, step by step, when you type a URL into your browser and hit Enter?

When I type a URL like `https://example.com/polls/`,
the browser reads `https` as the protocol,
`example.com` as the domain and `/polls/` as the path I want.
It uses DNS to look up the IP address for the domain if it doesn’t already have it.
It then connects to that address and, because the URL uses HTTPS, sets up an encrypted connection. 
The browser sends a GET request for `/polls/` and the server sends back a response. 
If it’s an HTML page, the browser reads it, requests any CSS, images or JavaScript the page needs, 
and displays it. When I ran my app locally, I used `127.0.0.1`, which is already an IP address, 
so the browser could skip the DNS lookup.

### What are the client and the server, and how do they talk to each other?

In my project, Chrome was the client and Django’s development server was the server. 
They were both running on my laptop. When I ran `python manage.py runserver`, 
Django started listening for requests on port `8000`. When I opened `http://127.0.0.1:8000/polls/`, 
Chrome sent an HTTP request to that port, and Django replied with the polls page.

### What is HTTP? What's inside a request and a response?

HTTP is the set of rules the browser and server use to understand each other’s messages. 
When I open a question, the request includes a method like `GET`, a path like `/polls/1/`, 
and headers with extra information, such as cookies. A request can also have a body. 
When I submit a vote, the body contains the choice I selected. The response has a status code that 
tells the browser what happened, headers that describe what came back, and usually a body. 
For a successful question page, the status is `200 OK` and the body contains the HTML that
Chrome displays.

### What's the difference between GET and POST, and when would you use each?

I use GET when I want to see something, like the list of polls or the choices for a question.
Opening or refreshing those pages should leave the vote count alone. 
I use POST when I submit the voting form because I’m sending Django my choice and asking it to 
record a vote. If voting just meant opening a GET link, even a search engine following that link 
could accidentally cast a vote.

### What do status codes like 200, 302, 404 and 500 mean?

I can see `200 OK` and `302 Found` at different points in the same vote. 
When I open a question, Django sends back its page with `200 OK`, which means the request worked. 
After I vote, Django responds with `302 Found`, a temporary redirect that tells Chrome to open the 
results URL. That’s why refreshing the results page just loads the results again.

`404 Not Found` means Django couldn’t find what I asked for. I got a 404 at `/` because I hadn’t 
set up a URL pattern for the home page. I also got one at `/polls/2/` because the pattern matched 
but there was no question with that ID. A `500 Internal Server Error` means something went wrong 
while the server was handling the request, for example if a view ran into a Python error it couldn’t 
handle.

### What jobs do HTML, CSS and JavaScript each do on a page?

On my voting page, HTML gives the page its structure and content. 
It puts the question on the page and creates the radio buttons and the Vote button. 
CSS controls how the page looks. In Part 6, I changed the links to green and added a background image 
by editing the CSS. JavaScript runs in the browser and can make the page respond to what I do. 
For example, it could update the results on screen without loading a new page. 
My voting form works without it because the browser can submit an HTML form on its own.

### How does a website remember who you are between pages?

I noticed this in the Django admin. After I logged in, I could open the Questions page without 
entering my password again. HTTP treats each request separately, so Django uses a session to connect 
those requests to the same login. It stores the session data on the server and sends Chrome a cookie 
containing a session ID. When I open the next admin page, Chrome sends the cookie back. Django uses 
that ID to find my session and see which user is logged in.

### In Django, how does a request travel through URLs, views, models and templates before a response comes back?

When I open `/polls/1/`, the request passes through Django’s middleware and reaches the URL patterns.
In `mysite/urls.py`, Django matches `polls/` and sends the remaining `1/` to `polls/urls.py`.
There, `1/` matches `<int:pk>/`, so Django calls `DetailView` with `1` as the question’s ID. 
The view uses the `Question` model to look up that question in the database. If it can’t find it,
I get a 404. If it finds it, the view gives the question to `detail.html`. 
The template fills in the question text and loops through the choices to create the radio buttons. 
Django then sends the finished HTML back to Chrome.

### Why did the tutorial add `{% csrf_token %}` to the voting form?

Another website could try to get my browser to submit a vote that I never chose. 
The browser might even include my cookies with that request. This is called cross-site request forgery,
or CSRF. Adding `{% csrf_token %}` puts a hidden field in the voting form. When I click Vote, Chrome 
sends the token along with my choice, and Django checks it before the vote is counted. Another site 
normally can’t read that token, so if it form sends a request with a missing or invalid token, Django
rejects it with `403 Forbidden`.### Why did the tutorial add {% csrf_token %} to the voting form?