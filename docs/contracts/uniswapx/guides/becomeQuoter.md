---
id: becomequoter
title: Become a Quoter
sidebar_position: 2
---
# Quoting During UniswapX Beta
To ensure a smooth swapping experience for traders during the beta period, the set of Quoters will be vetted by Uniswap Labs following UniswapX’s launch, with plans to make the quoting system fully permissionless in the near future.

Once you've been approved to be a quoter by the Uniswap Labs team follow the instructions below to complete your integration. If you have not been approved, please reach to the Uniswap Labs team [here](mailto:quoters@uniswap.org).

# Integrating with UniswapX RFQ

To participate as quoters, fillers must host a service that adheres to the UniswapX RFQ API schema (below) and responds to requests with quotes. The RFQ participant who submits the best quote for a given order will receive exclusive rights to fill it using their Executor during the _Exclusivity Period_ of the auction.

## Performance Expectations
During the UniswapX beta period, quoters will be expected to uphold the following standards to assure a fair auction process and the best experience for swappers. Any quoters who drop below these expectations are subject to suspension or removal from the UniswapX beta: 

1. **500ms Response Time:** When a quoter receives a request for quote, their server should respond within 500ms with either a quote for the trade or a 204 response code
2. **90% Rolling Fill Rate:** When a quoter wins an auction, meaning their contract address is in the `exclusiveFiller` field of an order, they are required to fill that order >90% of the time. We'll measure this on a rolling 7-day day period.

## RFQ API Schema

To successfully receive and respond to UniswapX RFQ Quotes, you must have a publicly accessible endpoint that receives incoming quote requests and responds with quotes by implementing the following schema:

Request:

```jsx
method: POST
content-type: application/json
data: {
    requestId: "string uuid - a unique identifier for swapper's request",
    tokenInChainId: "number - the `tokenIn` chainId",
    tokenOutChainId: "number - the `tokenOut` chainId",
    swapper: "string address - The swapper’s EOA address that will sign the order",
    tokenIn: "string address - The ERC20 token that the swapper will provide",
    tokenOut: "string address - The ERC20 token that the swapper will receive",
    amount: "string number - If the trade type is exact input then this is amount of `tokenIn` the user wants to swap otherwise this is amount of tokenOut the user wants to receive",
    type: "number - This is either `EXACT_INPUT` or `EXACT_OUTPUT`",
    quoteId: "string uuid - a unique identifier for the quote an integrator is sending back"
}
```

Response (status 200 - OK):

```jsx
{
    chainId: "number - the chainId for the quoted token",
    amountIn: "string number - If the request type is exact input then this field is `amount` from the quote request, otherwise this is the provided quote",
    amountOut: "string number - If the request type is exact output then this field is `amount` from the quote request, otherwise this is the provided quote",
    filler: "string address - The executor address that you would like to have last-look exclusivity for this order"

    { ...The following fields should be echoed from the quote request...},
    requestId: "string uuid - a unique identifier for this quote request",
    swapper: "string address - The swapper’s EOA address that will sign the order",
    tokenIn: "string address - The ERC20 token that the swapper will provide",
    tokenOut: "string address - The ERC20 token that the swapper will receive",
    quoteId: "string uuid - a unique identifier for the quote an integrator is sending back"
}
```

There is a latency requirement on responses from registered endpoints. Currently set to 500ms, but is subject to change. If you do not wish to respond to a quote request, you must return an empty response with status code `204`.

# (Optional) Signed Order Webhook Notifications

Signed open orders can always be fetched via the UniswapX API, but to provide improved latency there is the option to register for webhook notifications. Quoters can register an endpoint with their filler address, and receive notifications for every newly posted order that matches the filter. 

**Filter**

Orders can be filtered by various fields, but most relevant here is `filler`. When registering your webhook notification endpoint, we recommend you provide the `filler` address that you plan to use to execute orders and to receive the last-look exclusivity period. Alternatively the webhook can be configured to send all open orders to your endpoint. 

**Notification**

Order notifications will be sent to the registered endpoint as http requests as follows:

```jsx
method: POST
content-type: application/json
data: {
    orderHash: "the hash identifier for the order", 
    createdAt: "timestamp at which the order was posted",
    signature: "the swapper signature to include with order execution",
    orderStatus: "current order status (always should be `open` upon receiving notification)",
    encodedOrder: "The abi-encoded order to include with order execution. This can be decoded using the Uniswapx-SDK (https://github.com/uniswap/uniswapx-sdk) to verify order fields and signature",
    chainId: "The chain ID that the order originates from and must be settled on",
    filler?: "If this order was quoted by an RFQ participant then this will be their filler address",
    quoteId?: "If this order was quoted by an RFQ participant then this will be the requestId from the quote request",
    swapper: "OPTIONAL: the swapper address"
}
```

Once your quoting service is ready to receive quotes, message your Uniswap Labs contact to begin the onboarding process into UniswapX RFQ.