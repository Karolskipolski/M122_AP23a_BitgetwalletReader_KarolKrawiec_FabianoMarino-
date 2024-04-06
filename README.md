# Scriptlanguagesmodule
## Bitgetwallet project
In this project we will show you everything to our little Bitgetwalletreader.

### Systemdesign
![Screenshot 2024-04-06 103111](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/e2e86948-78ac-4a63-bb2d-33dd8293c4ac)

### Code
Here you can see the code that we have done:
'''Python

import requests
import hmac
import hashlib
import time
import json
import base64
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText

# Your API credentials
api_key = 'bg_c93715923a938f8e3d2711b65b7b7b4a'
secret_key = '7ab06f7afa2f0c4edaa868f2ea8c4b4eee678f41a9828f75565b5832eb3e27b4'
passphrase = 'BitcoinToTheMoon'

# Email credentials
sender_email = "krawieckarol9@gmail.com"
sender_password = "fkcj xcea zbya xack"
recipient_email = "krawieckarol9@gmail.com"

# API endpoint for balance inquiry
method = 'GET'
requestPath = '/api/spot/v1/account/assets'

# Function to generate timestamp
def generate_timestamp():
    return str(int(time.time() * 1000))

# Generate timestamp
timestamp = generate_timestamp()

# Creating the string to be signed
message = timestamp + method + requestPath

# Encrypting the string with HMAC SHA256
signature = hmac.new(secret_key.encode(), message.encode(), hashlib.sha256).digest()

# Base64 encoding the signature
signature_b64 = base64.b64encode(signature).decode()

# Preparing headers
headers = {
    'ACCESS-KEY': api_key,
    'ACCESS-SIGN': signature_b64,
    'ACCESS-TIMESTAMP': timestamp,
    'ACCESS-PASSPHRASE': passphrase,
    'Content-Type': 'application/json'
}

# Making the request
response = requests.get('https://api.bitget.com' + requestPath, headers=headers)

# Checking response
if response.status_code == 200:
    balance_info = json.dumps(response.json(), indent=4)

    # Email content
    subject = "Balance Information"
    body = balance_info

    # Create message container - the correct MIME type is multipart/alternative
    msg = MIMEMultipart()
    msg['From'] = sender_email
    msg['To'] = recipient_email
    msg['Subject'] = subject

    # Attach body to the email
    msg.attach(MIMEText(body, 'plain'))

    try:
        # Create SMTP session for sending the mail
        smtp_server = "smtp.gmail.com"
        smtp_port = 587
        server = smtplib.SMTP(smtp_server, smtp_port)
        server.starttls()
        # Login with your credentials
        server.login(sender_email, sender_password)
        # Send the message via the server
        server.sendmail(sender_email, recipient_email, msg.as_string())
        print("Email sent successfully!")
    except Exception as e:
        print("Failed to send email. Error:", str(e))
    finally:
        # Close the SMTP server session
        server.quit()
else:
    print("Error:", response.status_code, response.text)
'''
