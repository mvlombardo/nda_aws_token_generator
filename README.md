# nda_aws_token_generator

**Temporary AWS credentials (FederationUser Tokens) provided by the Data Manager service are being deprecated. Procedures for obtaining FederationUser Tokens using the new Package Service are available on [this wiki](https://github.com/NDAR/nda-data-access-example-code).**  

Libraries for using NDA web services to generate AWS tokens for accessing data

Requires NDA username, password, and server URL https://nda.nih.gov/DataManager/dataManager

Tokens expire every 24 hours, and the expiration date and time are included in the response from the web server in YYYY-MM-DDTHH:MM:SS-TZ format (TZ=HH:MM).

## Curl

- Bash implementation that uses curl to communicate with the Data Manager web service.

### Requirements

- sha1sum command line tool (GNU Tools)
- sed

### Example Usage

``` bash
ubuntu@ip-10-0-100-69:~/nda_aws_token_generator/curl$ bash generate_token.sh 'username' 'password' 'https://nda.nih.gov/DataManager/dataManager'

Beginning token request...
Access Key:    ASIAIL35Q7*REDACTED*
Secret Key:    OhFX4Ne+4mFD4gp9Bc07//FQ2bO6bN*REDACTED*
Session Token: AQoDYXdzEGQaoAIzZCKOuyJLDGzJXLR76hOEMXbbC0COP2J0slUaFxPQhZBWO6BG7VEfz6JteOodJkBAXQHS/h7SqJCDE2Jtu8ygYejvl8J8ykpYKWa3fhC+b0jxD5nuUbk/06wjfkYWBLNI1JoH1cskRBe0kxq9/ozIFnikcibjIWwTEYGHpGPydvEv1zc5eG0QAIaDP2RPePyK6DRLPHWINabOjV2drNQdi8r6CPndDNRFUvnyHnueuwEYuqIxaV4PqVkUaQvSGtLBVBaD/+pKNARJRMJxIQkeYeckBUBQArVxEpcQhqCMJ6dBwOMzi6XUkS4vMNhjIWlEmLN4Pb2BRZUSVZh1n78VICkD3CSx6tfjHuKHwC9HrXSmuJGtAiMYoMafZKUkTkogioy/sQU=
Expiration:    2015-10-28T02:44:26-04:00
```


## Python

### Requirements

All modules used are in standard library for python2 and python3.

### Example Usage

Install the NDA token generator library.

``` bash
cd ~/nda_aws_token_generator/python/
sudo python setup.py install
```
Create a python script that uses the library to generate tokens.

```
vim get_token_example.py
```

Copy and paste the following code into your 'get_token_example.py' script.

``` python
from nda_aws_token_generator import *
import getpass
import os

if sys.version_info[0] < 3:
    # Python 2 specific imports
    input = raw_input
    from ConfigParser import ConfigParser
else:
    # Python 3 specific imports
    from configparser import ConfigParser



web_service_url = 'https://nda.nih.gov/DataManager/dataManager'
username  = input('Enter your NIMH Data Archives username: ')
password  = getpass.getpass('Enter your NIMH Data Archives password: ')

generator = NDATokenGenerator(web_service_url)

token = generator.generate_token(username, password)

# Read .aws/credentials from the user's HOME directory, add a NDA profile, and update with credentials
parser = ConfigParser()
parser.read(os.path.expanduser('~/.aws/credentials'))

if not parser.has_section('NDA'):
    parser.add_section('NDA')
parser.set('NDA', 'aws_access_key_id', token.access_key)
parser.set('NDA', 'aws_secret_access_key', token.secret_key)
parser.set('NDA', 'aws_session_token', token.session)

with open (os.path.expanduser('~/.aws/credentials'), 'w') as configfile:
    parser.write(configfile)

print('aws_access_key_id=%s\n'
      'aws_secret_access_key=%s\n'
      'security_token=%s\n'
      'expiration=%s\n' 
      %(token.access_key,
        token.secret_key,
        token.session,
        token.expiration)
      )

```

Test running your script.

``` bash
ubuntu@ip-10-0-100-69:~/nda_aws_token_generator/python$ python get_token_example.py 

Enter your NIMH Data Archives username:username
Enter your NIMH Data Archives password:
aws_access_key_id=ASIAJOLANY*REDACTED*
aws_secret_access_key=eqT0n/ARR0U4uNDLrg2Qby55bzjexO*REDACTED*
security_token=AQoDYXdzEGQaoAKAG+uNIe8IqZuqJq2a1/SEv2NejCq8NERjxUnRQCOjJXwp2eH/g1lZGPQ1rdEKS5wP5QjvcsmrDU8JBrp5tLvv+IzWWwJ71u76/isRrMKtO8/LujaJHWW99UekZFQ5vS4moVt451bNggmK4+tq+naFuhyI0EkpqFB1PKfELFMhI7vpdt2oJltbs89oz5zHvCFiVoYGbPqtrdBMw8em9HmDqYRT+qdCpiUfN5ygWDlwFJS92tX5hzwmbUpz31190bdgcgfGhpAg/8sCWYEQutlgGr0kAX1nnEU0mmTn0mHEBQnQlsOPgtn1MjvgFaYxQluS2eVWIOc04n4ZCp9tVAU9dj5QcE6dflQZI4pw2da8Db1tDh2DnE5EybDOXg3xQNwg75S/sQU=
expiration=2015-10-28T03:03:11-04:00

```
# License

The MIT License (MIT)
Copyright (c) 2015 NIMH Data Archives

