This transaction process enables price negotiation on your marketplace. If you want to use this process in your Sharetribe Web Template, you will need to make the following modifications:

## 1. Push the process into Sharetribe using Sharetribe CLI
You can see the instructions in [this tutorial article](https://www.sharetribe.com/docs/tutorial/create-transaction-process/). Once you have pushed the process into your marketplace, you will be able to see it in [Console > Build > Transaction processes](https://console.sharetribe.com/transaction-processes/).

## 2. Configure the template to use this new process
There are a number of configurations in the template that are necessary for taking the new process into use.

- Add a new listing type in _src/config/configListing.js_, and add this negotiation process as the transaction type.
- Add a _transactionProcessNegotiation.js_ file to the _src/transactions_ folder. The file should describe the states and transitions of the process. You can model this file based on [this example](https://gist.github.com/SariSaar/e43af0af1fe9b5db43d32869efe229a5), or base it on _transactionProcessBooking.js_ in the same folder.
- Modify _transaction.js_ file in the same folder to return the negotiation process information when necessary
- Update _isBookingProcess_ function in _transaction.js_ to include this process.
- Add a _src/containers/TransactionPage/TransactionPage.stateDataNegotiation.js_ file to return the correct state information for each state of the process. You can model this file based on [this example](https://gist.github.com/SariSaar/ba3b0b63a0dbb571583c14c2d55d852e), or base it on _TransactionPage.stateDataBooking.js_ file in the same folder.
- Use _TransactionPage.stateDataNegotiation.js_ in the _TransactionPage.stateData.js_ file to return state data for the correct process.
- Add a _src/containers/InboxPage/InboxPage.stateDataNegotiation.js_ file to return the correct state information for each state of the process. You can model this file based on [this example](https://gist.github.com/SariSaar/73635736842daa735e6d93722ed35cdc), or base it on _InboxPage.stateDataBooking.js_ in the same folder.
- Use _InboxPage.stateDataNegotiation.js_ in the _InboxPage.stateData.js_ file to return state data for the correct process.
- Add process-specific microcopy to your microcopy file.

## 3. Add the correct inputs for price negotiation  

You will need to modify the user interface to handle the price negotiation process.

- Add a text input for a suggested total price on listing page, and show it for listings with negotiation as listing type.
- Add a price negotiation form element (text input + button) to the transaction page, so that when the participant has received an offer, they can make a counter offer.
- Show the action buttons for accepting and declining a transaction to the customer when the provider has made a counter offer.

## 4. Trigger the transaction directly from listing page

In the template, _CheckoutPage.js_ is configured to handle payments. Therefore, the simplest way to handle negotiation is to add a logic that triggers the transaction for the negotiation process directly from the listing page, and then redirects to _TransactionPage.js_. That way, the negotiation can happen on transaction page. You will need to trigger a transition each time one of the transaction participants makes an offer, and also when one of the participants accepts or declines the transaction.

Once the transaction price has been accepted, redirect the customer to _CheckoutPage.js_ to complete the payment. You will need to modify the `fnRequestPayment` function in `handlePaymentIntent` to trigger the correct transition for the negotiation process.


## 5. Calculate line items with the suggested total price
	
You will also need to modify the line item calculation used for the transaction. You will need to create a line item `suggested-adjustment` that reduces the total to the suggested number. The transaction process email templates expect a line item with code `line-item/suggested-adjustment`, so if your line item code is different, you will need to update the email templates as well. 

For instance, if the default transaction total would be $250 and the suggested total is $200, the `suggested-adjustment` line item would have quantity 1 and unitPrice -$50.

```js
// Returns the adjustment as negative if it is a discount, and 
// positive if it is an extra payment.
exports.resolveSuggestedAdjustment = (order, negotiatedTotal) => {
  if (!negotiatedTotal) {
    return null;
  }
  const orderTotalAmount = order.unitPrice.amount * order.quantity;
  return new Money(negotiatedTotal.amount - orderTotalAmount, negotiatedTotal.currency);
}
```

```js
  const suggestedAdjustment = resolveSuggestedAdjustment(order, orderData.negotiatedTotal)

  const suggestedAdjustmentLineItem = suggestedAdjustment ? [{
    code: 'line-item/suggested-adjustment',
    quantity: 1,
    unitPrice: suggestedAdjustment,
    includeFor: ['customer', 'provider'],
  }] : [];
```


## 6. Add relevant notifications
By default, the web template shows a provider notification when they have a transaction to accept. For the negotiation process, you will need to add a logic to _src/ducks/user.duck.js_ to fetch both customer and provider transactions where the next transitions require user action. You then need to add the notifications to _src/containers/TopbarContainer/TopbarContainer.js_ and _src/containers/InboxPage/InboxPage.js_. 

Customer action is required when the provider has made a counter offer, or when the provider has accepted the transaction and the customer needs to initiate manual payment. Provider action is needed when the customer has made an offer.