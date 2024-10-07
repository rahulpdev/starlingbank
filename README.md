![PyPI](https://img.shields.io/pypi/v/starlingbank.svg)

# starlingbank

An **unofficial** python package that provides access to parts of the Starling bank API. Designed to be used for personal use (i.e. using personal access tokens).

* [Change Log](#change-log)
* [Links](#links)
* [Installation](#installation)
* [Usage](#usage)
  * [Import](#import)
  * [Initialisation](#initialisation)
  * [Data](#data)
    * [Basic Account Data](#basic-account-data)
    * [Balance Data](#balance-data)
    * [Savings Goal Data](#savings-goal-data)
  * [Update a Single Savings Goal](#update-a-single-savings-goal)
  * [Add to / withdraw from a Savings Goal](#add-to--withdraw-from-a-savings-goal)
  * [Download a Savings Goal Image](#download-a-savings-goal-image)


## Change Log
7/10/2024
* Added SpendingCategory class and relevant method to StarlingAccount class.

18/12/2019
* Removed `available_to_spend` property as this is no longer supported by the API.

23/02/2018
* Updated to use v2 API.
* `currency` is no longer a property of the balance data.
* `id` and `name` are no longer properties of the account data.
* `account_number` is now `account_identifier`.
* `sort_code` is now `bank_identifier`.
* An API call is now made when initialising a StarlingAccount instance, even with `update=False`. This is to get the minimum data needed to start working with an account.

## Links

* https://www.starlingbank.com/
* https://developer.starlingbank.com/

## Installation
```shell
pip install starlingbank
```

## Usage
### API Key Scope Requirements
To use all of the functionality this package affords, the following API scopes are required:

* account:read
* account-identifier:read
* balance:read
* savings-goal:read
* savings-goal-transfer:read
* savings-goal-transfer:create
* transaction:read

### Import
```python
from starlingbank import StarlingAccount
```

### Initialisation
```python
my_account = StarlingAccount("<INSERT API TOKEN HERE>")
```
If using a sandbox token:
```python
my_account = StarlingAccount("<INSERT API TOKEN HERE>", sandbox=True)
```
By default, to save on wasted API calls only minimal data is collected when you initialise a StarlingAccount. To optionally update all data on initialisation use the following:
```python
my_account = StarlingAccount("<INSERT API TOKEN HERE>", update=True)
```

### Data
4 data sets are currently supported:

1. Basic Account Data
2. Balance Data
3. Savings Goal Data
4. Spending Category Data

 You have to request / refresh each set of data as required with the following commands:

```python
my_account.update_account_data()
my_account.update_balance_data()
my_account.update_savings_goal_data()
my_account.update_spending_categories_data(month, year)
```
_Note: Available values for month are: JANUARY, FEBRUARY, MARCH, APRIL, MAY, JUNE, JULY, AUGUST, SEPTEMBER, OCTOBER, 
NOVEMBER, DECEMBER. Omission of the month or year parameter will return data for the current month._

#### Basic Account Data
Properties:

* account_identifier
* bank_identifier
* currency
* iban
* bic
* created_at

Example:
```python
print(my_account.account_identifier)
```

#### Balance Data
Properties:

* cleared_balance
* effective_balance
* pending_transactions
* accepted_overdraft

Example:

```python
print(my_account.effective_balance)
```

#### Savings Goal Data
Savings goals are stored as a dictionary of objects where the dictionary key is the savings goals uid. To get a list of savings goal names and their respective uids you can run:

```python
for uid, goal in my_account.savings_goals.items():
    print("{0} = {1}".format(uid, goal.name))
```

Each goal has the following properties:

* uid
* name
* target_currency
* target_minor_units
* total_saved_currency
* total_saved_minor_units

_Note: Values are in minor units e.g. £1.50 will be represented as 150._

Example:
```python
print(my_account.savings_goals['c8553fd8-8260-65a6-885a-e0cb45691512'].total_saved_minor_units)
```

#### Spending Category Data
Spending categories are stored as a dictionary of SpendingCategory objects, where the dictionary key is the spending_category. To get a list of spending categories and their respective data, you can run:

```python
for category, spending in my_account.spending_categories.items():
    print("{0}: Total Spent = {1}, Total Received = {2}".format(category, spending.total_spent, spending.total_received))
```

Each category has the following properties:

* spending_category
* net_direction
* currency
* total_spent
* total_received
* net_spend
* percentage
* transaction_count

Example:

```python
print(my_account.spending_categories['PAYMENTS'].total_spent)
```

### Update a Single Savings Goal
The `update_savings_goal_data()` method will update all savings goals but you can also update them individually with the following method:
```python
my_account.savings_goals['c8553fd8-8260-65a6-885a-e0cb45691512'].update()
```

### Add to / withdraw from a Savings Goal
You can add funds to a savings goal with the followng method:
```python
my_account.savings_goals['c8553fd8-8260-65a6-885a-e0cb45691512'].deposit(1000)
```

You can add funds to a savings goal with the followng method:
```python
my_account.savings_goals['c8553fd8-8260-65a6-885a-e0cb45691512'].withdraw(1000)
```

_Note: Both methods above will call an update after the transfer so that the local total_saved value is correct._

### Download a Savings Goal Image
You can download the image associated with a savings goal to file with the following method:
```python
my_account.savings_goals['c8553fd8-8260-65a6-885a-e0cb45691512'].get_image('<YOUR CHOSEN FILENAME>.png')
```
_Note: If the filename is ommitted the name of the goal will be used. You can optionally specify a full path alongside the filename if required._