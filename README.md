# Scriptlanguagesmodule
## Bitgetwallet project
In this project we will show you everything to our little Bitgetwalletreader.

## Systemdesign
![Screenshot 2024-04-06 103111](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/e2e86948-78ac-4a63-bb2d-33dd8293c4ac)

## Code
Here you can see the code that we have done:
```
# BitgetWalletReader
# This program shows the assets on my Bitget spot account
# Karol Krawiec, Fabiano Marino
# v1.0, 08.04.24

import requests
import hmac
import hashlib
import time
import json
import base64
import smtplib
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
import schedule

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

# Function to send email
def send_email():
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

# Schedule the email to be sent every 5 minutes
schedule.every(5).minutes.do(send_email)

# Infinite loop to run the scheduler
while True:
    schedule.run_pending()
    time.sleep(1)

```
so what this code does is that it searches up my Bitgetwallets spotaccount and then gives me my balance of every single coin in JSON format via mail.

But because it is a lot of code, we should break it up so you can understand more of it.

### Imports
So as you can see we have some imports so lets go through every single one of them:
```
import requests
```
This library allows you to send HTTP requests easily.
```
import hmac
```
This module provides functions for generating keyed hashes of messages using the HMAC algorithm.
```
import hashlib
```
This module provides a common interface to many secure hash and message digest algorithms.
```
import time
```
This module provides various time-related functions.
```
import json
```
This module allows you to work with JSON (JavaScript Object Notation) data.
```
import base64
```
This module provides functions to encode and decode data using Base64 encoding.
```
import smtplib
```
This module defines an SMTP client session object that can be used to send mail to any internet machine with an SMTP or ESMTP listener daemon.
```
from email.mime.multipart import MIMEMultipart
from email.mime.text import MIMEText
```
These are classes from the email.mime module that allow you to create multipart email messages with text and attachments.
```
import schedule
```
This import works kind of like cronjob. You can set a timer in it.

### API credentials
#### Quick reminder!
These credentials look different in my code and we have to use these placeholders for safety reasons.
As you can see in the code you have to put in your API-key, secret-key and passphrase. We were really struggeling with getting this right but eventually we did.
```
api_key = '_______________________________'
secret_key = '_______________________________________'
passphrase = '________________'
```
![Screenshot 2024-04-06 123624](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/c1cfe4b4-a75e-4c8a-9a1b-f2a264a48261)

Here is also an image of what I had to do on bitget. This is an API-adress I created.
### Email config
In this part we have set up the email which recives the mail and the one which sends it. We also had to manage the security part on google and had to get a code which alows us to use our gmail in this script.
```
sender_email = "krawieckarol9@gmail.com"
sender_password = "fkcj xcea zbya xack"
recipient_email = "krawieckarol9@gmail.com"
```

### API endpoint
With this next part we are getting the endpoint of my assets. We got this from bitget it self, because they are offering alot of these endpoint to people who programm trading bots.
```
method = 'GET'
requestPath = '/api/spot/v1/account/assets'
```

### Timestamp
In the next lines we configured the timestamp that it matches the current balance. We have to do this because of the fact that the balance can change anytime because of the market.
```
# Function to generate timestamp
def generate_timestamp():
    return str(int(time.time() * 1000))

# Generate timestamp
timestamp = generate_timestamp()

# Creating the string to be signed
message = timestamp + method + requestPath

# Encrypting the string with HMAC SHA256
signature = hmac.new(secret_key.encode(), message.encode(), hashlib.sha256).digest()
```
In these steps we are also preparing the signature to send as a request to the server.

### Signature
In this step we are doing the signature which we will use to get the needed access to my wallet.
```
signature_b64 = base64.b64encode(signature).decode()

headers = {
    'ACCESS-KEY': api_key,
    'ACCESS-SIGN': signature_b64,
    'ACCESS-TIMESTAMP': timestamp,
    'ACCESS-PASSPHRASE': passphrase,
    'Content-Type': 'application/json'
}
```
We are also preparing the headers in JSON format so its readable for the server.

### Response
Now in this step we are making a response to the server and afterwards it checks if you have acces or not, if you do then it will say in the console: "Email sent successfully" and gives you an Email.
```
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
```

### Schedule (cronjob)
This is the last part of the script and we used the schedule to make the program send an email every 5 minutes.
```
# Schedule the email to be sent every 5 minutes
schedule.every(5).minutes.do(send_email)

# Infinite loop to run the scheduler
while True:
    schedule.run_pending()
    time.sleep(1)
```
Now this was the quick runthrough of the code lets look at the preparation next.

## Preparation for running the script
### Config RasberryPi to internet
If you have a working VM or a RasberryPi then you can run this script from there. In our case we used a RasberryPi. You have to plug it in via LAN and also to some source of electricity. After the light is shining you can check if it is configured through a IP scanner.

![Screenshot 2024-04-06 133721](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/0f0d583c-bf34-4a91-8a37-5f8e613ec35b)

As you can see we are connected.

### CMD
Now open cmd and type in:
```
ssh pi@karol.home
```

![Screenshot 2024-04-06 134231](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/70a4208c-238b-42e8-bad2-bd77459622ff)

Make sure you use the name that you got from the IP scanner.
Now there will be a field where it asks you for your password so put it in.

![Screenshot 2024-04-06 134244](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/e001f455-86dc-464a-bf2a-08a2137b1a13)

Afterwards you should be ready to go!

### Config the timestamp
Now before we run the script we have to configure the timestamp so we dont get errors. We will do it by giving the RasberryPi these two commands:
```
sudo apt-get install ntp
sudo service ntp restart
```

![Screenshot 2024-04-06 134551](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/de982046-ac0a-4763-af42-d03bba6468fe)

This will configure the timestamp how it should be.

### Running the script
Now we are ready to run the script. You have use this command:

´´´
python BitgetWallet.py
´´´

if you do not have the file yet, create one and go further.
Now since you have runned the script, you can see that the email was sent successfully.

![Screenshot 2024-04-06 134932](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/b1edaff5-c986-4fd2-9422-b43bb8701255)

Then you will get an email and youre good to go!

![Screenshot 2024-04-06 134947](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/eaecd808-8a7c-4216-8658-afdb5d37a099)

![Screenshot 2024-04-06 134959](https://github.com/Karolskipolski/BitgetWalletScriptproject/assets/142780585/19a30068-9f45-429f-bec6-77a3884e7521)
