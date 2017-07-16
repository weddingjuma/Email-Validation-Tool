<p align="center">
<img style='width:220px;' src='http://i.imgur.com/gZ5RBFV.png' alt='PHP Email Validation Tool' />
</p>

[![codecov](https://codecov.io/gh/daveearley/EmailValidationTool/branch/master/graph/badge.svg)](https://codecov.io/gh/daveearley/EmailValidationTool) [![Build Status](https://travis-ci.org/daveearley/EmailValidationTool.svg?branch=master)](https://travis-ci.org/daveearley/EmailValidationTool) [![Code Climate](https://codeclimate.com/github/daveearley/EmailValidationTool/badges/gpa.svg)](https://codeclimate.com/github/daveearley/EmailValidationTool)

**An extensible email validation library for PHP 7+**

The aim of this library is to offer a more detailed email validation report than simply checking if an email is the valid format, and also to make it possible to easily add custom validations. Currently this tool checks the following:


| Validation  | Description |
| ------------- | ------------- |
| MX records  | Checks if the email's domain has valid MX records  |
| Valid format  | Validates e-mail addresses against the syntax in RFC 822, with the exceptions that comments and whitespace folding and dotless domain names are not supported (as it uses PHP's filter_var().  |
| Email Host  | Checks if the email's host (e.g gmail.com) is reachable  |
| Role/Business Email^  | Checks if the email if a role/business based email (e.g info@reddit.com).  |
| Disposable email provider^  | Checks if the email is a disposable email (e.g person@10minutemail.com).  |
| Free email provider^  | Checks if the email is a free email (e.g person@yahoo.com).  |
| Misspelled Email ^ | Checks the email for possible typs and returns a suggested corrected (e.g hi@gmaol.con -> hi@gmail.com).  |

^ **Date used for these checks can be found [here](https://github.com/daveearley/Email-Validation-Tool/tree/master/src/data)**

# Installation

```bash
composer require daveearley/daves-email-validation-tool
```

# Usage
## Quick Start

```php
// Include the compose autoload file
require __DIR__ . '/vendor/autoload.php';

$validator = EmailValidation\EmailValidatorFactory::create('dave@gmoil.con');

$jsonResult = $validator->getValidationResults()->asJson();
$arrayResult = $validator->getValidationResults()->asAray();

echo $jsonResult;

```

Expected output:

```json
{
"valid_format": true,
"valid_mx_records": false,
"possible_email_correction": "dave@gmail.com",
"free_email_provider": false,
"disposable_email_provider": false,
"role_or_business_email": false,
"valid_host": false
}
```

## Adding Custom Validations

To add a custom validation simply extend the [EmailValidation\Validations\Validator](https://github.com/daveearley/Email-Validation-Tool/blob/master/src/Validations/Validator.php) class and implement the **getResultResponse()** and **getValidatorName()** methods. You then register the validation using the **EmailValidation\EmailValidator->registerValidator()** method.


### Example code

// Validations/GmailValidator.php
```php
<?php

namespace EmailValidation\Validations;

class GmailValidator extends Validator
{
    public function getValidatorName(): string
    {
        return 'is_gmail';
    }

    public function getResultResponse(): bool
    {
        $hostName = $this->getEmailAddress()->getHostPart();
        return strpos($hostName, 'gmail.com') !== false;
    }
}
```

// file-where-you-doing-your-validation.php
```php
<?php

use EmailValidation\Validations\GmailValidator;

require __DIR__ . '/vendor/autoload.php';

$validator = EmailValidation\EmailValidatorFactory::create('dave@gmail.com');

$validator->registerValidator(new GmailValidator());

echo $validator->getValidationResults()->asJson();
```

The expected output will be:

```json
{
"is_gmail": true,
"valid_format": true,
"valid_mx_records": false,
"possible_email_correction": "",
"free_email_provider": true,
"disposable_email_provider": false,
"role_or_business_email": false,
"valid_host": false
}
```

## Running in Docker
You can eaily validate over HTTP using docker. To get docker working run 
```bash
docker-compose up -d 
```
in the repository root. You can then validate an email by navigating to http://localhost:8880?email=email.to.validate@example.com. The result will be JSON string as per above.

# FAQ

### Is this validation accurate?
No, none of these tests are 100% accurate. As with any email validation there will always be false positives & negatives. The only way to guarantee an email is valid is to send an email and solicit a response. However, this library is still useful for detecting disposable emails etc., and also acts as a good first line of defence.