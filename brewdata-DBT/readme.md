# BrewData Strategy Documentation

## Standard Strategies

| ID | Strategy Name | Semantic Group |
|----|--------------|----------------|
| 1 | Address (Random) | address |
| 2 | Street Name (Random) | address |
| 3 | City (Random) | address |
| 4 | State (Random) | address |
| 5 | Person Name (Random) | name |
| 6 | First Name (Random) | name |
| 7 | Last Name (Random) | name |
| 8 | Date (yyyy-mm-dd) | date |
| 9 | Phone Number (Random) | phone_number |
| 10 | Phone Number (By CountryCode) | phone_number |
| 11 | License Plate (Random) | license_plate |
| 12 | SSN (Random) | national_id |
| 13 | SAID (Random) | national_id |
| 14 | Job (Random) | job |
| 15 | Personal Email (Random) | email |
| 16 | Company Email (Random) | email |
| 17 | IPv4 Address (Random) | ip_address |
| 18 | Domain Name (Random) | domain_name |
| 19 | Device MAC Address (Random) | device_address |
| 20 | Credit Card Number (Random) | credit_card |
| 21 | Bank Account (Random) | bank_account |
| 22 | Swift Code (Random) | swift_code |
| 23 | Random Number | random_number |
| 24 | Random Letters | random_letters |
| 25 | Country (Random) | address |
| 26 | Checksum | checksum |
| 27 | Barcode (Random) | barcode |
| 28 | Color (Random) | color |
| 29 | Date Time (yyyy-mm-dd hh:mm:ss) | date |
| 30 | Land Coordinates (Random) | land_coordinates |
| 32 | Gender (Random) | gender |
| 33 | Date Number (yyyymmdd) | date |
| 34 | Building Number (Random) | address |
| 35 | Address Line1 (Random) | address |
| 36 | Address Line2 (Random) | address |
| 37 | Company Name (Random) | company |
| 38 | Postal Code (Random) | address |
| 39 | Zipcode (Random) | address |
| 40 | Credit Card Type (Random) | credit_card |
| 41 | Credit Card Expiry Date (Random) | credit_card |
| 44 | Year (yyyy) | date |
| 45 | Month Number (mm) | date |
| 46 | Day Number (dd) | date |
| 47 | RegEx Pattern | random_letters |
| 48 | State Code (Random) | address |
| 49 | Telephone Number (Random) | phone_number |
| 50 | Address (by Distance) | address |
| 51 | Street Name (by City) | address |
| 52 | City (by State/Province) | address |
| 53 | Province (Random) | address |
| 54 | Person Name (Retain Gender) | name |
| 55 | First Name (Retain Gender) | name |
| 56 | Last Name (Retain Religion, Nativity) | name |
| 85 | Random Alpha Numeric | random_alpha_numeric |
| 86 | IPv6 Address (Random) | ip_address |
| 87 | Retain | retain |

## GAN Strategies
These strategies use different GAN models for synthetic data generation. The appropriate strategy (categorical/numeric) is applied based on the column type.

| ID | Strategy Name | Type |
|----|--------------|------|
| 59 | CTGAN Neural Network | Categorical |
| 60 | CTGAN Neural Network | Numeric |
| 61 | TVAE Neural Network | Categorical |
| 62 | TVAE Neural Network | Numeric |
| 63 | CopulaGAN Neural Network | Categorical |
| 64 | CopulaGAN Neural Network | Numeric |
| 65 | Transformer Neural Network | Categorical |
| 66 | Transformer Neural Network | Numeric |
| 67 | NoGAN | Categorical |
| 68 | NoGAN | Numeric |



