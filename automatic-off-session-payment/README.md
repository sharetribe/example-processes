This transaction process enables automatic off-session payments on your marketplace. If you want to use this process in your Sharetribe Web Template, you will need to make the following modifications:


## 1. Push the process into your Flex marketplace using Flex CLI
You can see the instructions in [this tutorial article](https://www.sharetribe.com/docs/tutorial/create-transaction-process/). Once you have pushed the process into your marketplace, you will be able to see it in [Flex Console > Build > Transaction processes](https://flex-console.sharetribe.com/transaction-processes/).


## 2. Configure the template to use this new process

There are a number of configurations in the template that are necessary for taking the new process into use.

- Add a new listing type in _src/config/configListing.js_, and add your automatic off-session payment as the transaction type.
- Add a _transactionProcessOffSession.js_ file to _src/transactions_ folder. The file should describe the states and transitions of the process. You can model this file after [this example](https://gist.github.com/SariSaar/1425bb6dcbec1f78d7f4701cff8c4e7e), or according to _transactionProcessBooking.js_ in the same folder.
- Modify _transaction.js_ file in the same folder to return the off-session process information when necessary
- Add a _src/containers/TransactionPage/TransactionPage.stateDataOffSession.js_ file to return the correct state information for each state of the process. You can model this file after [this example](https://gist.github.com/SariSaar/fe77a93b3892e4fbe266f180c0fbc98b), or according to _TransactionPage.stateDataBooking.js_ in the same folder.
- Use _TransactionPage.stateDataOffSession.js_ in the _TransactionPage.stateData.js_ file to return state data for the correct process.
- Add a _src/containers/InboxPage/InboxPage.stateDataOffSession.js_ file to return the correct state information for each state of the process. You can model this file after [this example](https://gist.github.com/SariSaar/d74334ed48338b68ecd2f8c6cfa0b9a2), or according to _InboxPage.stateDataBooking.js_ in the same folder.
- Use _InboxPage.stateDataOffSession.js_ in _InboxPage.stateData.js_ file to return state data for the correct process.

## 3. Trigger the transaction directly from listing page

In the template, _CheckoutPage.js_ is configured to handle payments. However, in the off-session process, the payment intent is not created upon booking. Therefore, the simplest way to handle the off-session flow is to add a logic that triggers the transaction for the off-session process directly from the listing page, and then redirects to _TransactionPage_.

If the automatic payment does not happen for some reason (e.g. insufficient funds or missing payment method), redirect the customer from _TransactionPage_ to _CheckoutPage_ and modify the `fnRequestPayment` function in `handlePaymentIntent` to trigger the correct transition for the automatic off-session payment process.


## 4. Add relevant notifications

By default, the FTW template shows a provider notification when they have a transaction to accept. For the automatic off-session payment process, you will need to add a logic to _src/ducks/user.duck.js_ to fetch both customer and provider transactions where the next transitions require user action. You then need to add the notifications to _src/containers/TopbarContainer/TopbarContainer.js_ and _src/containers/InboxPage/InboxPage.js_. Customer action is required when the automatic payment fails and the customer needs to update their payment method and trigger the payment manually.