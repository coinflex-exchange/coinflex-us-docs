# Change Log


# Introduction

Welcome to CoinFLEX US's application programming interface (API). CoinFLEX US's APIs provide clients programmatic access to control aspects of their accounts and to place orders on CoinFLEX US's trading platform. CoinFLEX US supports the following types of APIs:

* a WebSocket API
* a REST API

Using these interfaces it is possible to place both authenticated and unauthenticated API commands for public and prvate commands respectively.

To get started please register for a TEST account at `https://stg.coinflex.us/user-console/register`


# API Key Management

An API key is required to make an authenticated API command.  API keys (public and corresponding secret key) can be generated via the CoinFLEX US GUI within a clients account. 

By default, API Keys are read-only and can only read basic account information, such as positions, orders, and trades. They cannot be used to trade such as placing, modifying or cancelling orders.

If you wish to execute orders with your API Key, clients must select the `Can Trade` permission upon API key creation.

API keys are also only bound to a single sub-account, defined upon creation. This means that an API key will only ever interact and return account information for a single sub-account.

# Historical Data

CoinFLEX US's historical L2 order book data (depth data), can be found at [https://docs.tardis.dev/historical-data-details/coinflexus] (https://docs.tardis.dev/historical-data-details/coinflexus) (third party API), which includes historical market data details - instruments, data coverage and data collection specifics for all of our instruments since 2020-07-14.

# Rate Limit

CoinFLEX US's APIs allows our clients to access and control their accounts or view our market data using custom-written software. To protect the performance of the system, we impose certain limits:

Type                              |                            Limit |
--------------------------------- | -------------------------------- |
Rest API                          |                   100 per second |
Rest API                          |                  2500 per 5 mins |
Initialising Websocket Connection |                   200 per minute |
Websocket API (Auth)              |                    50 per second |
Websocket API (No Auth)           |                     1 per second |
