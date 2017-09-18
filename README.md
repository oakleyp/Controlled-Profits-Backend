# Controlled Profits Backend

Rails JSON API for controlled profits site

All responses are formatted as specified by JSONAPI.org v1.0, with the exception of the Content-Type Header, which is ignored at the time of writing.

http://jsonapi.org/format/

## Authentication

All authentication is done using JSON Web Tokens. After creating a user using the request method and routes listed below, you must request a token, which will be given in the header of /auth/sign_in response if the posted user's credentials are correct. This header will contain these important fields:

access-token → vrLn7nm6kNIbAaJ_AIlyQQ

client → xpjUiiBvu9Ssb4oelpOBjA

token-type → Bearer

uid → user_email@email.com

Following that sign in response, these fields must be provided in every subsequent request sent to the server in order to be authenticated. If you do not already have a solution to manage this part of the request cycle, I have read that JToker might be a good option.

## Routes:

### User Routes:

**NOTE: All of these paths are preceded by `/auth/`**

| path | method | purpose |
|:-----|:-------|:--------|
| /    | POST   | Email registration. Requires **`firstname`**, **`lastname`**, **`email`**, **`password`**, and **`password_confirmation`** params. Allows **`active_business_id`**|
| / | DELETE | Account deletion. This route will destroy users identified by their **`uid`**, **`access_token`** and **`client`** headers. |
| / | PUT/PATCH | Account updates. This route will update an existing user's account settings. Accepted params are **`firstname`**, **`lastname`**, **`active_business_id`**, **`password`** and **`password_confirmation`**. If **`config.check_current_password_before_update`** is set to `:attributes` the **`current_password`** param is checked before any update, if it is set to `:password` the **`current_password`** param is checked only if the request updates user password. |
| /sign_in | POST | Email authentication. Requires **`email`** and **`password`** as params. This route will return a JSON representation of the `User` model on successful login along with the `access-token` and `client` in the header of the response. |
| /sign_out | DELETE | Use this route to end the user's current session. This route will invalidate the user's authentication token. You must pass in **`uid`**, **`client`**, and **`access-token`** in the request headers. |
| /:provider | GET | Set this route as the destination for client authentication. Ideally this will happen in an external window or popup. [Read more](#omniauth-authentication). |
| /:provider/callback | GET/POST | Destination for the oauth2 provider's callback uri. `postMessage` events containing the authenticated user's data will be sent back to the main client window from this page. [Read more](#omniauth-authentication). |
| /validate_token | GET | Use this route to validate tokens on return visits to the client. Requires **`uid`**, **`client`**, and **`access-token`** as params. These values should correspond to the columns in your `User` table of the same names. |
| /password | POST | Use this route to send a password reset confirmation email to users that registered by email. Accepts **`email`** and **`redirect_url`** as params. The user matching the `email` param will be sent instructions on how to reset their password. `redirect_url` is the url to which the user will be redirected after visiting the link contained in the email. |
| /password | PUT | Use this route to change users' passwords. Requires **`password`** and **`password_confirmation`** as params. This route is only valid for users that registered by email (OAuth2 users will receive an error). It also checks **`current_password`** if **`config.check_current_password_before_update`** is not set `false` (disabled by default). |
| /password/edit | GET | Verify user by password reset token. This route is the destination URL for password reset confirmation. This route must contain **`reset_password_token`** and **`redirect_url`** params. These values will be set automatically by the confirmation email that is generated by the password reset request. |

### Business Routes:

**NOTE: All of these paths are preceded by `/v1/businesses/`**
**Also, all request headers must include the fields described in the `Authentication` section**

| path | method | purpose |
|:-----|:-------|:--------|
| / | GET    | Returns a JSON array listing all a current user's businesses|
| / | POST   | Creates a new business. Requires fields **`name`**, **`naics`**, **`sic`**, and **`ein`** |
| /:bid | GET | Returns a JSON object with all of a given businesses' stored details, where :bid is the business id |
| /:bid | PATCH/PUT | Updates a business by ID |
| /:bid | DELETE    | Deletes a business by ID |

### Business Data Entry Routes:

**NOTE: All of these paths are preceded by `/v1/businesses/:bid/data`**
**Also, all request headers must include the fields described in the `Authentication` section**

| path | method | purpose |
|:-----|:-------|:--------|
| / | GET    | Returns a JSON array listing the entire history of a given business' data entries |

#### Optional request parameters (Join as many as necessary in the URL with the & symbol)
| path | method | purpose |
|:-----|:-------|:--------|
| /?start_date=yyyy/mm/dd&end_date=yyyy/mm/dd) | GET | Returns all of a businesses data entries, within a specified date range |
| /?section=(income_statement/balance_sheet/sales_and_marketing/finanacial_roi) | GET | Returns only one section for each businesses data entry |
| /?entry_type=(actual/adjusted) | GET | Returns data entries by entry type: actual or adjusted |

#### JSON Response format:
```
{
  data: [
    {
      id: "1",
      type: "business_data_entry",
      entry_type: "actual",
      entry_date: "2017-09-30T19:34:55.000Z",
      business_id: "5",
      income_statement: {...},
      balance_sheet: {...},
      sales_and_marketing: {...},
      financial_roi: {...}
    }, 
    {
      ...
    },
    ...
  ]
}
```

For example, a `GET` request to the url:

`http://localhost:3000/v1/businesses/1/data?section=income_statement&entry_type=actual&start_date=2017/09/03&end_date=2017/09/06`

might return this response:
```
{
    "data": [
        {
            "id": 3,
            "type": "business_data_entry",
            "entry_type": "actual",
            "entry_date": "2017-09-03T19:34:55.000Z",
            "business_id": 1,
            "income_statement": {
                "period_sales": "9.99",
                "cash_collections": "9.99",
                "credit_sales": "9.99",
                "cogs": "9.99",
                "marketing": "9.99",
                "direct_labor": "9.99",
                "distribution": "9.99",
                "vpie": "9.99",
                "salaries": "9.99",
                "benefit_admin": "9.99",
                "office_lease": "9.99",
                "office_supplies": "9.99",
                "utilities": "9.99",
                "transportation": "9.99",
                "online_expenses": "9.99",
                "insurance": "9.99",
                "training": "9.99",
                "accounting_and_legal": "9.99",
                "advertising": "9.99",
                "marketing_development": "9.99",
                "other": "9.99",
                "fpie": "9.99",
                "interest_paid": "9.99",
                "depreciation_and_amortizaton": "9.99",
                "donations": "0",
                "tax_rate": "9.99",
                "dividends": "9.99",
                "entry_date": "2017-09-03T19:34:55.000Z"
            }
        }
    ]
}
```

Complete response example given entry_type=actual:

```
{
    "data": [
        {
            "id": 1,
            "type": "business_data_entry",
            "entry_type": "actual",
            "entry_date": "2017-09-30T23:59:59.999Z",
            "business_id": 1,
            "income_statement": {
                "total_revenues": "1397500.0",
                "credit_sales": "50000.0",
                "cogs": "594000.0",
                "marketing": "38250.0",
                "direct_labor": "15000.0",
                "distribution": "18000.0",
                "vpie": null,
                "salaries": "120000.0",
                "benefit_admin": "18972.0",
                "office_lease": "12000.0",
                "office_supplies": "1000.0",
                "utilities": "5400.0",
                "transportation": "6800.0",
                "online_expenses": "2700.0",
                "insurance": "2448.0",
                "training": "6000.0",
                "accounting_and_legal": "2400.0",
                "advertising": "1000.0",
                "marketing_development": "500.0",
                "other": "0.0",
                "fpie": "0.0",
                "interest_paid": "3500.0",
                "depreciation_and_amortization": "2500.0",
                "donations": 0,
                "tax_rate": "0.34",
                "dividends": "50000.0",
                "entry_date": "2017-09-30T23:59:59.999Z"
            },
            "balance_sheet": {
                "cash": "125000.0",
                "accounts_receivable": "110000.0",
                "inventory": "88000.0",
                "prepaid_expenses": "25000.0",
                "other_current_assets": "10000.0",
                "ppe": "15000.0",
                "furniture_and_fixtures": "10000.0",
                "leasehold_improvements": "1000.0",
                "land_and_buildings": "167000.0",
                "other_fixed_assets": "44000.0",
                "accumulated_depreciation": "25000.0",
                "goodwill": "280000.0",
                "accounts_payable": "50000.0",
                "interest_payable": null,
                "taxes_payable": "15000.0",
                "deferred_revenue": "22000.0",
                "short_term_notes": "50000.0",
                "current_debt": "50000.0",
                "other_current_liabilities": "38000.0",
                "bank_loans_payable": "90000.0",
                "notes_payable_to_stockholders": "75000.0",
                "other_long_term_debt": "38000.0",
                "common_stock": "44000.0",
                "paid_in_surplus": "0.0",
                "retained_earnings": "253000.0",
                "entry_date": "2017-09-30T23:59:59.999Z"
            },
            "sales_and_marketing": {
                "prospects": "50000.0",
                "number_of_sales": "10200.0",
                "marketing_spend": "38250.0",
                "grand_total_units": "14500.0",
                "entry_date": "2017-09-30T23:59:59.999Z"
            },
            "financial_roi": {
                "airp_debt": "0.08",
                "airp_equity": "0.04",
                "airc_for_financing": "0.12",
                "corp_tax_rate": "0.34",
                "entry_date": "2017-09-30T23:59:59.999Z"
            }
        }
    ]
}
```

#### Creating a new business data entry:

**NOTE: entry_date will automatically be stored as the last day at the end of the month, and is not a required field**

POST to `/v1/businesses/:bid/data/` with the following fields: (have fun)
  :total_revenues,
  :credit_sales,
  :cogs,
  :marketing,
  :direct_labor,
  :distribution,
  :vpie,
  :salaries,
  :benefit_admin,
  :office_lease,
  :office_supplies,
  :utilities,
  :transportation,
  :online_expenses,
  :insurance,
  :training,
  :accounting_and_legal,
  :advertising,
  :marketing_development,
  :other,
  :fpie,
  :ebitda,
  :interest_paid,
  :depreciation_and_amortization,
  :tax_rate,
  :dividends,
  :cash,
  :accounts_receivable,
  :inventory,
  :prepaid_expenses,
  :other_current_assets,
  :ppe,
  :furniture_and_fixtures,
  :leasehold_improvements,
  :land_and_buildings,
  :other_fixed_assets,
  :accumulated_depreciation,
  :goodwill,
  :accounts_payable,
  :interests_payable,
  :taxes_payable,
  :deferred_revenue,
  :short_term_notes,
  :current_debt,
  :other_current_liabilities,
  :bank_loans_payable,
  :notes_payable_to_stockholders,
  :other_long_term_debt,
  :common_stock,
  :paid_in_surplus,
  :retained_earnings,
  :prospects,
  :number_of_sales,
  :marketing_spend,
  :grand_total_units,
  :airp_debt,
  :airp_equity,
  :airc_for_financing,
  :corp_tax_rate

### Profit Drivers Routes 

**NOTE: All of these paths are preceded by `/v1/businesses/:bid/profit_drivers/`**
**Also, all request headers must include the fields described in the `Authentication` section**  

| path | method | purpose |
|:-----|:-------|:--------|
| / | GET | Returns all of the current business' profit driver values for the current month |
| / | POST | Creates a new profit driver entry for the current month, given the JSON request as laid out below |

#### Optional request parameters (Join as many as necessary in the URL with the & symbol)
| path | method | purpose |
|:-----|:-------|:--------|
| /?start_date=yyyy/mm/dd&end_date=yyyy/mm/dd) | GET | Returns a business' profit driver inputs within the specified date range |

* More parameters may be added as necessary

----------------------------------------------

#### Creating a new profit driver entry

**NOTE: All JSON POST request headers must contain the field `Content-Type: application/json`**
**NOTE: entry_date will automatically be stored as the last day at the end of the month, and is not a required field**

  Example body of `POST` request to `http://localhost:3000/v1/businesses/1/profit_drivers` :
```
  {
	"data": {
		"type": "profit_drivers_data",
		"profit_drivers": {
			"prospects": {
				"percent": 0.01,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"conversions": {
				"percent": 0.15,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"volume": {
				"percent": 0.05,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"price": {
				"percent": 0.03,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"productivity": {
				"percent": 0.01,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"efficiency": {
				"percent": 0.01,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			},
			"frequency": {
				"percent": 0.07,
				"var_cost": 100.0,
				"fixed_cost": 100.0
			}
		}
	}
}
```

Individually selected profit drivers can also be updated or created within the same month:
```
  {
    "data": {
      "type": "profit_drivers_data",
      "profit_drivers": {
        "frequency": {
          "percent": 0.07,
          "var_cost": 100.0,
          "fixed_cost": 100.0
        },
        "productivity": {
          "percent": 0.01,
          "var_cost": 100.0,
          "fixed_cost": 100.0
        }
      }
    }
  }
```

----------------------------------------------

#### Querying profit driver entries

Example response from `GET` request to 
`http://localhost:3000/v1/businesses/1/profit_drivers/?start_date=2017/09/030&end_date=2017/09/31` :

```
{
    "data": {
        "type": "profit_drivers_data",
        "profit_drivers": {
            "conversions": {
                "percent": "0.15",
                "var_cost": "100.0",
                "fixed_cost": "100.0"
            },
            "volume": {
                "percent": "0.05",
                "var_cost": "100.0",
                "fixed_cost": "100.0"
            },
            ...,
            ...
        },
        "entry_date": "2017-09-30T23:59:59.999Z"
    }
}
```