This transaction process allows your customers to insantly book listings, bypassing the default preauthorization step where the provider needs to either accept or decline the booking. This process also supports push payment methods through the request-push-payment transition. Note that the instructions below do not enable push payments in your client application.

For an instant booking transaction, you completely remove the preauthorized state. The confirm-payment transition goes directly to the accepted state. 

## 1. Push this process into your Flex marketplace using Flex CLI

You can see the instructions in [this tutorial article](https://www.sharetribe.com/docs/tutorial/create-transaction-process/). Once you have pushed the process into your marketplace, you will be able to see it in [Flex Console > Build > Transaction processes](https://flex-console.sharetribe.com/transaction-processes/).

## 2. Configure the template to use this new process

There are a number of configurations in the template that are necessary for taking the new process into use.


1. Add a new listing type in [src/config/configListing.js](https://github.com/sharetribe/web-template/blob/main/src/config/configListing.js), and add this process as the transaction type:

    ```js
    {
        listingType: 'instant-booking',
        label: 'Instant booking',
        transactionType: {
        process: 'instant-booking',
        alias: 'instant-booking/release-1',
        unitType: 'day',
        }
    }
    ```

2. Add a transactionInstantBooking.js file to the src/transactions folder. The file should describe the states and transitions of the process. You can model this file based on [this example](https://gist.github.com/shareoc/cecbc95d3200602854f54fb30a596589), or base it on [transactionProcessBooking.js](https://github.com/sharetribe/web-template/blob/main/src/transactions/transactionProcessBooking.js) in the same folder.

3. Modify transaction.js file in the same folder. 

    The file should import the new process created in the previous step:
    ```js
    import * as instantBookingProcess from './transactionProcessInstantBooking';
    ```

    Define the booking process name:
    ```js 
    export const INSTANT_BOOKING_PROCESS_NAME = 'instant-booking';```

    Export the process:
    ```js
    {
        name: INSTANT_BOOKING_PROCESS_NAME,
        alias: `${INSTANT_BOOKING_PROCESS_NAME}/release-1`,
        process: instantBookingProcess,
        unitTypes: [DAY, NIGHT, HOUR],
    },
    ```

    And modify the isBookingProcess function to include the instant booking process:

    ```js
    export const isBookingProcess = processName => {
    const latestProcessName = resolveLatestProcessName(processName);
    const processInfo = PROCESSES.find(process => process.name === latestProcessName);
    return [BOOKING_PROCESS_NAME].includes(processInfo?.name);
    return [BOOKING_PROCESS_NAME, INSTANT_BOOKING_PROCESS_NAME].includes(processInfo?.name);
    };
    ```

4. Add a src/containers/TransactionPage/TransactionPage.stateDataInstantBooking.js file to return the correct state information for each state of the process. You can model this file based on [this example](https://gist.github.com/shareoc/b8c0b1cdecf53b25ac158ffae5676d6a), or base it on [TransactionPage.stateDataBooking.js](https://github.com/sharetribe/web-template/blob/main/src/containers/TransactionPage/TransactionPage.stateDataBooking.js) file in the same folder.

5. Modify the TransactionPage.stateData.js file to return the right state data:

    Import the instant booking process name:

    ```js
    import {
    BOOKING_PROCESS_NAME,
    PRODUCT_PROCESS_NAME,
    INSTANT_BOOKING_PROCESS_NAME,
    resolveLatestProcessName,
    } from '../../transactions/transaction';
    ```

    Import the state data from the file created in the previous step:

    ```js 
    import { getStateDataForInstantBookingProcess } from './TransactionPage.stateDataInstantBooking.js';```

    Add a condition to the end of the file that returns the correct state data:

    ```js
    else if (processName === INSTANT_BOOKING_PROCESS_NAME) {
        return getStateDataForInstantBookingProcess(params, processInfo());
    }
    ```

6. Add a src/containers/InboxPage/InboxPage.stateDataInstantBooking.js file to return the correct state information for each state of the process. You can model this file based on [this example](https://gist.github.com/shareoc/eb018f18c087b158de77c0c71327a50b), or base it on [InboxPage.stateDataBooking.js](https://github.com/sharetribe/web-template/blob/main/src/containers/InboxPage/InboxPage.stateDataBooking.js) in the same folder.

7. Modify the [inboxPage.stateData.js](https://github.com/sharetribe/web-template/blob/main/src/containers/InboxPage/InboxPage.stateData.js) file to return the right state data:

    Import the process name:

    ```js
    import {
    BOOKING_PROCESS_NAME,
    INSTANT_BOOKING_PROCESS_NAME,
    PRODUCT_PROCESS_NAME,
    resolveLatestProcessName,
    getProcess,
    } from '../../transactions/transaction';
    ```

    Import the file created in the previous step:

    ```js 
    import { getStateDataForInstantBookingProcess } from './InboxPage.stateDataInstantBooking.js';```

    Add a condition to the end of the file to return the right state data:

    ```js
    else if (processName === INSTANT_BOOKING_PROCESS_NAME) {
        return getStateDataForInstantBookingProcess(params, processInfo());
    } 
    ```

8. Add process-specific microcopy to your microcopy file. Here are some examples:

    ```js
    "BookingDatesForm.instantBook": "Book",
    "CheckoutPage.instant-booking.title": "Complete booking",
    "CheckoutPage.instant-booking.orderBreakdown": "Order breakdown",
    "InboxPage.instant-booking.booked.status": "Booked",
    "TransactionPage.ActivityFeed.instant-booking.booked": "The bicycle was booked by {otherUsersName}",
    "TransactionPage.instant-booking.customer.booked.title": "Your booking was succesful!",
    "TransactionPage.instant-booking.provider.booked.title": "A booking was made by {customerName}",
    "TransactionPanel.instant-booking.orderBreakdownTitle": "Order breakdown",
    ```


## 3. Show different translations for the instant booking process in the client application

The instant booking process should work without the following modifications, but you will probably want to show the user different translations based on different transaction processes. In addition to adding new translation keys to the microcopy file, you will need to conditionally render the translations in the BookingDatesForm and StripePaymentForm components. 

1. Pass the `transactionProcessAlias` as a prop to the BookingDatesForm component in [OrderPanel.js](https://github.com/sharetribe/web-template/blob/main/src/components/OrderPanel/OrderPanel.js#L236-L253): `transactionProcessAlias={transactionProcessAlias}`

2. Using the `transactionProcessAlias` and `INSTANT_BOOKING_PROCESS_NAME` variables in the [BookingDatesForm.js](https://github.com/sharetribe/web-template/blob/main/src/components/OrderPanel/BookingDatesForm/BookingDatesForm.js) file you can render different translations for the instant booking process.
` const isInstantBooking = transactionProcessAlias.includes(INSTANT_BOOKING_PROCESS_NAME);`

3. Pass the [StripePaymentForm](https://github.com/sharetribe/web-template/blob/main/src/containers/CheckoutPage/StripePaymentForm/StripePaymentForm.js) component the following prop:
`isInstantBooking={transactionProcessAlias.includes(INSTANT_BOOKING_PROCESS_NAME)}`
and modify the `isBookingYesNo` ternary:  `const isBookingYesNo = isBooking ? isInstantBooking ? 'no' : 'yes' : 'no';`
