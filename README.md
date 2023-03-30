# socialFIND
socialFIND is a social media username checker tool that checks if a given username exists on various social media platforms.

## How the script works?

- The tool takes a single argument "username" and optionally a flag "-d" or "--debug" to enable debug mode.

- The script uses the "requests" module to make HTTP requests to the social media platforms' URLs, and the "json" module to load the social media URLs and error messages from the "socialFIND.json" file.

- The script first checks if the given username contains a period "." character, which is not allowed in some social media platforms' usernames. If it does, the script skips checking that platform.

- The script then loops through the social media URLs defined in the "socialFIND.json" file, substituting the username into each URL, and making a GET request with headers that include a User-Agent string to simulate a web browser request.

- If the URL returns a response with an HTTP status code other than 404, or the expected error message is not found in the response text, the script prints a message indicating that the username exists on that social media platform and writes the URL to a file named "username.txt".

- If the URL returns a response with an HTTP status code of 404 or the expected error message is found in the response text, the script prints a message indicating that the username does not exist on that social media platform.

- If there is an error during the request, the script prints an error message with the error type and message.

- Finally, the script prints a message indicating that the results have been saved to a file named "username.txt".

## Requirements

Install your libraries:
```bash
pip3 install requests, json, argparse 
```

## Permissions

Ensure you give the script permissions to execute. Do the following from the terminal:
```bash
sudo chmod +x socialFIND.py
```

## Usage

Help:

```bash
sudo python3 socialFIND.py -h                                                                              ─╯
usage: socialFIND.py [-h] [-d] username

positional arguments:
  username     check the services with the given username

options:
  -h, --help   show this help message and exit
  -d, --debug  enable debug mode
```

Basic:

```bash
sudo python3 socialFIND.py -d [username]
```

## Example script
```python
# -*- coding: utf-8 -*-
# Author : Dimitrios Zacharopoulos
# All copyrights to Obipixel Ltd
# 30 March 2023

#!/usr/bin/env python3

import requests
import json
import os
import sys 
import argparse

DEBUG = False


def write_to_file(url , filename):
    with open(filename , "a") as f:
        f.write(url+"\n")

def print_error(err  , errstr , var , debug = False):
    if debug:
        print (f"\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {errstr}\033[93;1m {err}")
    else:
        print (f"\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {errstr}\033[93;1m {var}")

def make_request(url, headers, error_type, social_network):
    try:
        r = requests.get(url, headers=headers)
        if r.status_code:
            return r, error_type
    except requests.exceptions.HTTPError as errh:
        print_error(errh, "HTTP Error:", social_network, DEBUG)
    except requests.exceptions.ConnectionError as errc:
        print_error(errc, "Error Connecting:", social_network, DEBUG)
    except requests.exceptions.Timeout as errt:
        print_error(errt, "Timeout Error:", social_network, DEBUG)
    except requests.exceptions.RequestException as err:
        print_error(err, "Unknown error:", social_network, DEBUG)
        return None, ""


def socialFIND(username):
    # Not sure why, but the banner messes up if i put into one print function
    print("┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┓┃┏━━━┓━━┓━┓┃┏┓━━━┓")
    print("┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┏━━┛┫┣┛┃┗┓┃┃┓┏┓┃")
    print("┏━━┓━━┓━━┓┓━━┓┃┃┃┃┗━━┓┃┃┃┏┓┗┛┃┃┃┃┃")
    print("┃━━┫┏┓┃┏━┛┫┃┓┃┃┃┃┃┏━━┛┃┃┃┃┗┓┃┃┃┃┃┃")
    print("┣━━┃┗┛┃┗━┓┃┗┛┗┓┗┓┛┗┓┃┃┫┣┓┃┃┃┃┃┛┗┛┃")
    print("┗━━┛━━┛━━┛┛━━━┛━┛━━┛┃┃━━┛┛┃┗━┛━━━┛")
    print("┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃")
    print("┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃")
    print()

    fname = username+".txt"

    if os.path.isfile(fname):
        os.remove(fname)
        print("\033[1;92m[\033[0m\033[1;77m*\033[0m\033[1;92m] Removing the previous file:\033[1;37m {}\033[0m".format(fname))

    print("\033[1;92m[\033[0m\033[1;77m*\033[0m\033[1;92m] Checking the username\033[0m\033[1;37m {}\033[0m\033[1;92m on: \033[0m".format(username))
    raw = open("socialFIND.json", "r", encoding="utf-8")
    data = json.load(raw)

    # User agent is needed because some sites does not 
    # return the correct information because it thinks that
    # we are bot
    headers = {
        'User-Agent': 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10.15; rv:109.0) Gecko/20100101 Firefox/111.0'
    }

    for social_network in data:
        url = data.get(social_network).get("url").format(username)
        error_type = data.get(social_network).get("errorType")
        cant_have_period = data.get(social_network).get("noPeriod")

        if ("." in username) and (cant_have_period == "True"):
            print("\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {}:\033[93;1m The User Name is Not Allowed!".format(social_network))
            continue
            
        r, error_type = make_request(url=url, headers=headers, error_type=error_type, social_network=social_network)
        
        if error_type == "message":
            error = data.get(social_network).get("errorMsg")
            # Checks if the error message is in the HTML
            if not error in r.text:
                print("\033[37;1m[\033[91;1m+\033[37;1m]\033[91;1m {}:\033[0m".format(social_network), url)
                write_to_file(url, fname)                   
            
            else:
                print("\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {}:\033[93;1m Not on here!".format(social_network))
            
        elif error_type == "status_code":
            # Checks if the status code of the repsonse is 404
            if not r.status_code == 404:
                print("\033[37;1m[\033[91;1m+\033[37;1m]\033[91;1m {}:\033[0m".format(social_network), url)
                write_to_file(url, fname)
            
            else:
                print("\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {}:\033[93;1m Not on here!".format(social_network))

        elif error_type == "response_url":
            error = data.get(social_network).get("errorUrl")
            # Checks if the redirect url is the same as the one defined in socialFIND.json
            if not error in r.url:
                print("\033[37;1m[\033[91;1m+\033[37;1m]\033[91;1m {}:\033[0m".format(social_network), url)
                write_to_file(url, fname)
            else:
                print("\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {}:\033[93;1m Not on here!".format(social_network))

        elif error_type == "":
            print("\033[37;1m[\033[91;1m-\033[37;1m]\033[91;1m {}:\033[93;1m Error!".format(social_network))

    print("\033[1;92m[\033[0m\033[1;77m*\033[0m\033[1;92m] Saved: \033[37;1m{}\033[0m".format(username+".txt"))

if __name__ == "__main__":
    parser = argparse.ArgumentParser()
    parser.add_argument('username', help='check the services with the given username')
    parser.add_argument("-d", '--debug', help="enable debug mode", action="store_true")

    args = parser.parse_args()
    
    if args.debug:
        DEBUG = True

    if args.username:
        socialFIND(args.username)
```

## Example output

```
sudo python3 socialFIND.py -d obipixel_

┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┓┃┏━━━┓━━┓━┓┃┏┓━━━┓
┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┏━━┛┫┣┛┃┗┓┃┃┓┏┓┃
┏━━┓━━┓━━┓┓━━┓┃┃┃┃┗━━┓┃┃┃┏┓┗┛┃┃┃┃┃
┃━━┫┏┓┃┏━┛┫┃┓┃┃┃┃┃┏━━┛┃┃┃┃┗┓┃┃┃┃┃┃
┣━━┃┗┛┃┗━┓┃┗┛┗┓┗┓┛┗┓┃┃┫┣┓┃┃┃┃┃┛┗┛┃
┗━━┛━━┛━━┛┛━━━┛━┛━━┛┃┃━━┛┛┃┗━┛━━━┛
┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃
┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃┃

[*] Removing the previous file: obipixel_.txt
[*] Checking the username obipixel_ on:
[+] Instagram: https://www.instagram.com/obipixel_
[+] Twitter: https://www.twitter.com/obipixel_
[+] Facebook: https://www.facebook.com/obipixel_
[+] YouTube: https://www.youtube.com/obipixel_
[-] Blogger: Not on here!
[+] Google Plus: https://plus.google.com/+obipixel_
```

## License Information

This library is released under the [Creative Commons ShareAlike 4.0 International license](https://creativecommons.org/licenses/by-sa/4.0/). You are welcome to use this library for commercial purposes. For attribution, we ask that when you begin to use our code, you email us with a link to the product being created and/or sold. We want bragging rights that we helped (in a very small part) to create your 9th world wonder. We would like the opportunity to feature your work on our homepage.
