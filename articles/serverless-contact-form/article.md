#---
title: The Simplest Way to Add a Contact Form to a Static HTML site
description: Add a contact form to a static HTML site quickly, easily and essentially for free using
a serverless function on GCP, AWS, or Azure.

tags: serverless, python, html, JavaScript
---
# Setting Up Your Contact Form
## What we'll need:
* A GCP (or other cloud service) account
* A SendGrid account
### SendGrid
* Create an account
* Verify a domain or single sender
* Get an API Key
### GCP
* Create an account
* Create a cloud function
    * Make it Public
    * Add the SendGrid API key as an Env Var
### The Code for the Function
```python
import logging
import os

from sendgrid import SendGridAPIClient
from sendgrid.helpers.mail import Mail

YOUR_EMAIL_ADDRESS = "you@example.com"


def trigger_contact_form(request):
    """
    This is the entrypoint for our GCP function.
     
    It will verify the request JSON then try to send the mail.
    """
    request_json = request.get_json()
    if error := validate_incoming_json(request_json):
        return error, 400
    successful = send_mail(request_json['message'],
                           request_json['sender_email'],
                           request_json['sender_name'])
    if successful:
        return dict(success=True), 202
    else:
        return dict(success=False, error="Unknown error"), 500


def validate_incoming_json(request_json):
    """
    Given a request.json, ensure that it has all of the keys we expect.
    """
    if not request_json:
        return dict(error="No JSON present")
    for potential_key in ('email', 'name', 'message'):
        if potential_key not in request_json:
            return dict(error=f'missing JSON field "{potential_key}')
    # Could also do lots of other things here: regex to validate emails for example
    # In General, it's probably best not to handle this kind of serialisation yourself
    # Something like Marshmallow is a good choice
    # But for this article, we've kept it simple.


def send_mail(email_message: str, sender_email: str, sender_name: str) -> bool:
    """
    Given a recipient, message, and sender, send the email. Return True/False depending on success.
    """
    body = "\n".join((f"Sender Email Address: {sender_email}",
                      f"Sender Name: {sender_name}",
                      f"\nSender Message:\n{email_message}"))
    message = Mail(
        from_email=os.environ['SENDER_EMAIL'],
        to_emails=YOUR_EMAIL_ADDRESS,
        subject='New Contact Form Message',
        plain_text_content=body)
    sg = SendGridAPIClient(os.environ.get('SENDGRID_API_KEY'))
    try:
        response = sg.send(message)
    except Exception as e:
        logging.exception(e)
        return False
    if response.status_code != 202:
        logging.error("An email didn't send correctly", response)
        return False
    return True

```
```requirements.txt
sendgrid
```
### The Code for the Form
#### HTML
```html
<!--TODO: Get Owen to add this in the most minimal format possible-->
```
#### JavaScript
```javascript
// TODO get Owen to add this in the most minimal format possible
```
