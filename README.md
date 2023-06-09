CrackQL
=======
CrackQL is a GraphQL password brute-force and fuzzing utility.

<h1 align="center">
	<img src="https://github.com/nicholasaleks/CrackQL/blob/master/static/CrackQL-Banner.png?raw=true" alt="CrackQL"/>
	<br>
</h1>

CrackQL is a versatile GraphQL penetration testing tool that exploits poor rate-limit and cost analysis controls to brute-force credentials and fuzz operations.

# Table of Contents
* [How it works](#how-it-works)
* [Attack Use Cases](#attack-use-cases)
  * [Defense Evasion](#defense-evasion)
  * [Password Spraying Brute-forcing](#password-spraying-brute-forcing)
  * [Two-factor Authentication OTP Bypass](#two-factor-Authentication-otp-bypass)
  * [User Account Enumeration](#user-account-enumeration)
  * [Insecure Direct Object Reference](#insecure-direct-object-reference)
  * [General Fuzzing](#general-fuzzing)
* [Inputs](#inputs)
* [Installation](#installation)
* [Configurations](#configuration)
* [Maintainers](#maintainers)
* [Mentions](#mentions)

## How it works?

CrackQL works by automatically batching a single GraphQL query or mutation into several alias operations. It determines the number of aliases to use based on the CSV input variables. After programmatically generating the batched GraphQL document, CrackQL then batches and sends the payload(s) to the target GraphQL API and parses the results and errors.

## Attack Use Cases

CrackQL can be used for a wide range of GraphQL attacks since it programmatically generates payloads based on a list of dynamic inputs.

### Defense Evasion

Unlike [Burp Intruder](https://portswigger.net/burp/documentation/desktop/tools/intruder) which sends a request for each unique payload, CrackQL evades traditional API HTTP rate-limit monitoring defenses by using multiple alias queries to stuff large sets of credentials into single HTTP requests. To bypass query cost analysis defenses, CrackQL can be optimized into using a series of smaller batched operations (`-b`) as well as a time delay (`-D`).


### Password Spraying Brute-forcing

CrackQL is perfect against GraphQL deployments that leverage in-band GraphQL authentication operations (such as the [GraphQL Authentication Module](https://www.graphql-modules.com/docs#authentication-module)). The below password spraying example works against [DVGA](https://github.com/dolevf/Damn-Vulnerable-GraphQL-Application) with the `sample-inputs/users-and-passwords.csv` dictionary.

[sample-queries/login.graphql](sample-queries/login.graphql)
<pre>
mutation {
  login(username: <b>{{username|str}}</b>, password: <b>{{password|str}}</b>) {
    accessToken
  }
}
</pre>

### Two-factor Authentication OTP Bypass

It is possible to use CrackQL to bypass two-factor authentication by sending all OTP (One Time Password) tokens

[sample-queries/otp-bypass.graphql](sample-queries/otp-bypass.graphql)
<pre>
mutation {
  twoFactor(otp: <b>{{otp|int}}</b>) {
    accessToken
  }
}
</pre>

### User Account Enumeration

CrackQL can also be used for enumeration attacks to discover valid user ids, usernames and email addresses

[sample-queries/enumeration.graphql](sample-queries/enumeration.graphql)
<pre>
query {
  signup(email: <b>{{email|str}}</b>, password: <b>{{password|str}}</b>) {
    user {
      email
    }
  }
}
</pre>

### Insecure Direct Object Reference

CrackQL could be used to iterate over a large number of potential unique identifiers in order to leak object information

[sample-queries/idor.graphql](sample-queries/idor.graphql)
<pre>
query {
  profile(uuid: <b>{{uuid|int}}</b>) {
    name
    email
    picture
  }
}
</pre>

### General Fuzzing

CrackQL can be used for general input fuzzing operations, such as sending potential SQLi and XSS payloads.

## Inputs

CrackQL will generate payloads based on input variables defined by a CSV file. CrackQL requires the CSV header to match the input name.

[sample-inputs/usernames_and_passwords.csv](sample-inputs/usernames_and_passwords.csv)
<pre>
<b>username</b>, <b>password</b>
admin, admin
admin, password
admin, pass
admin, pass123
admin, password123
operator, operator
operator, password
operator, pass
operator, pass123
operator, password123
</pre>

#### Valid input types
- `str`
- `int`
- `float`

## Installation

### Requirements
- Python3
- Requests
- GraphQL
- Jinja

### Clone Repository
```
git clone git@github.com:nicholasaleks/CrackQL.git
```

### Get Dependencies
`pip install -r requirements.txt`

### Run CrackQL
`python3 CrackQL.py -h`

```
Usage: python3 CrackQL.py -t http://example.com/graphql -q sample-queries/login.graphql -i sample-inputs/usernames_and_passwords.csv

Options:
  -h, --help            show this help message and exit
  -t URL, --target=URL  Target url with a path to the GraphQL endpoint
  -q QUERY, --query=QUERY
                        Input query or mutation operation with variable
                        payload markers
  -i INPUT_CSV, --input-csv=INPUT_CSV
                        Path to a csv list of arguments (i.e. usernames,
                        emails, ids, passwords, otp_tokens, etc.)
  -d DELIMITER, --delimiter=DELIMITER
                        CSV input delimiter (default: ",")
  -o OUTPUT_DIRECTORY, --output-directory=OUTPUT_DIRECTORY
                        Output directory to store results (default:
                        ./results/[domain]_[uuid]/
  -b BATCH_SIZE, --batch-size=BATCH_SIZE
                        Number of batch operations per GraphQL document
                        request (default: 100)
  -D DELAY, --delay=DELAY
                        Time delay in seconds between batch requests (default:
                        0)
  --verbose             Prints out verbose messaging
  -v, --version         Print out the current version and exit.
```

## Configuration
Use `config.py` to set HTTP cookies, headers or proxies if the endpoint requires authentication.

## Maintainers
* [Nick Aleks](https://github.com/nicholasaleks)
* [Dolev Farhi](https://github.com/dolevf)

## Mentions
* [Kitploit](https://www.kitploit.com/2022/07/crackql-graphql-password-brute-force.html)
