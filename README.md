# Sharetribe Flex example transaction processes

This repository contains example transaction processes and the email
templates for [Sharetribe
Flex](https://www.sharetribe.com/flex/). These processes can be used
as starting points for customizing your marketplace using Flex CLI.

## Prerequisites

To understand the transaction engine in Sharetribe Flex, see the
[Transaction process
article](https://www.sharetribe.com/docs/background/transaction-process/)
in Flex Docs.

To get up and running with Flex CLI, see the [Getting started with
Flex
CLI](https://www.sharetribe.com/docs/flex-cli/getting-started-with-flex-cli/)
guide in Flex Docs.

## Usage

Clone this repository:

```
git clone git@github.com:sharetribe/flex-example-processes.git
```

Change to the cloned directory:

```
cd flex-example-processes
```

Let's use `my-marketplace-id` as an example Marketplace ID and
`instant-booking` as the process that is taken into use.

Using the example processes, create a new process to your marketplace:

```
flex-cli process create -m my-marketplace-id --process instant-booking --path instant-booking
```

Create an alias to the process:

```
flex-cli process create-alias -m my-marketplace-id --process instant-booking --version 1 --alias release-1
```

Check that everything is good:

```
flex-cli process list -m my-marketplace-id --process instant-booking
```

Then set up your Sharetribe Web Web Template customization to use that process alias and
continue customizing your process and UI. You can find process-specific suggestions in the README.md file of each process.

## Learn more

For customizing the transaction process, see the [transaction process
format
guide](https://www.sharetribe.com/docs/references/transaction-process-format/).

For editing the transactional email templates, see the [Email
templates
reference](https://www.sharetribe.com/docs/references/email-templates/)
in Flex Docs.

## Processes

The example processes are meant to showcase some of the capabilities
of the transaction engine. See the differences in the `process.edn`
files in each process directory to see how they differ only slightly.

All the processes support [Strong Customer Authentication
(SCA)](https://www.sharetribe.com/docs/background/strong-customer-authentication/).

The example processes differ mostly in availability management and
pricing. To understand these concepts, see the [Listing availability
management](https://www.sharetribe.com/docs/references/availability/)
and [Custom
pricing](https://www.sharetribe.com/docs/background/custom-pricing/)
articles in Flex Docs.

### default-booking:

This is the default process that is created in our backend for new
test marketplaces and used for bookings.

When used in **Sharetribe Web Template** ([web-template](https://github.com/sharetribe/web-template/)) customizing
[pricing](https://www.sharetribe.com/docs/concepts/pricing/) can be
done within the template by utilizing [privileged
transitions](https://www.sharetribe.com/docs/concepts/privileged-transitions/).

![default-booking](./default-booking.png)

### default-purchase:

This is the default process that is created in our backend for new
test marketplaces and used for purchases. It is a
transaction process designed for product selling with shipping or pickup, and it
uses the Flex stock management features.

When used in **Sharetribe Web Template** ([web-template](https://github.com/sharetribe/web-template/)) customizing
[pricing](https://www.sharetribe.com/docs/concepts/pricing/) can be
done within the template by utilizing [privileged
transitions](https://www.sharetribe.com/docs/concepts/privileged-transitions/).

![default-purchase](./default-purchase.png)

### default-inquiry:

This is the default process that is created in our backend for new test marketplaces and used for free messaging. Its only function is to start a transaction so participants can send messages within the context of that transaction. The process does not use any payment, booking, or stock features.

![default-inquiry](./default-inquiry.png)

### instant-booking:

This is an example of instant booking supporting both card and push payment methods.

See [payment methods overview](https://www.sharetribe.com/docs/concepts/payment-methods-overview/) for more info.

The [README.md](./instant-booking/README.md) file in the process folder describes how to take the process into use in the Sharetribe Web Template.

![instant-booking](./instant-booking.png)


### negotiated-booking:

This process differs clearly from the other processes as seen in the
visualization below. It uses price negotiation where the customer and
provider can negotiate a new total price for the transaction.

The transitions for the negotiation showcase an example how to handle
the negotiation in an offering phase before moving onto the payment.

The [README.md](./negotiated-booking/README.md) file in the process folder describes how to take the process into use in the Sharetribe Web Template.

Note that price negotiation is just one way to customize the
pricing. Our powerful [Custom
pricing](https://www.sharetribe.com/docs/concepts/custom-pricing/) enables several use cases for transaction pricing.

In addition to price negotiation, you can also add a functionality to negotiate the booking days within the process. For this use case, you would need to add the [`update-booking` action](https://www.sharetribe.com/docs/references/transaction-process-actions/#actionupdate-booking) to the negotiation transitions, and pass the new suggested booking times as transition parameters. To do this, you would need to make sure in your client app that the listing has availability for the suggested new booking time.

![negotiated-booking](./negotiated-booking.png)

### automatic-off-session-payment

This process makes it possible to create bookings for more than 90 days in the future and still use the default Stripe integration. The automatic off-session payment process creates the booking when the transaction is initiated, but handles the payment closer to the actual booking time. See the [automatic off-session payments article](https://www.sharetribe.com/docs/concepts/off-session-payments-in-transaction-process/) for more information.

The [README.md](./automatic-off-session-payment/README.md) file in the process folder describes how to take the process into use in the Sharetribe Web Template.

![automatic-off-session-payment](./automatic-off-session-payment.png)