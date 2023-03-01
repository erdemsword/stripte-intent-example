
'stripe_payment' package to your project's pubspec.yaml file:
```
dependencies:
  stripe_payment: ^1.1.4
```

Here's an example implementation:

```
import 'package:flutter/material.dart';
import 'package:stripe_payment/stripe_payment.dart';

class PaymentWidget extends StatefulWidget {
  final double estimatedCost;
  final Function(double) onPaymentSuccess;

  PaymentWidget({required this.estimatedCost, required this.onPaymentSuccess});

  @override
  _PaymentWidgetState createState() => _PaymentWidgetState();
}

class _PaymentWidgetState extends State<PaymentWidget> {
  bool _isProcessingPayment = false;
  double _finalAmount = 0.0;

  void _handlePayment() async {
    StripePayment.setOptions(
        StripeOptions(publishableKey: 'YOUR_PUBLISHABLE_KEY_HERE'));

    PaymentMethod paymentMethod =
        await StripePayment.createPaymentMethod(PaymentMethodRequest(
      card: CreditCard(
        number: '4242424242424242',
        expMonth: 12,
        expYear: 24,
        cvc: '123',
      ),
    ));

    if (paymentMethod.id != null) {
      setState(() {
        _isProcessingPayment = true;
      });

      // Calculate the final amount to be charged (estimated cost + 20%)
      _finalAmount = widget.estimatedCost * 1.2;

      PaymentIntentResult paymentIntentResult =
          await StripePayment.confirmPaymentIntent(PaymentIntent(
        paymentMethodId: paymentMethod.id!,
        currency: 'USD',
        amount: (_finalAmount * 100).toInt(),
        confirmationMethod: PaymentIntentConfirmationMethod.manual,
        confirm: true,
      ));

      setState(() {
        _isProcessingPayment = false;
      });

      if (paymentIntentResult.status == 'succeeded') {
        widget.onPaymentSuccess(_finalAmount);
      } else {
        // Payment failed, handle error
      }
    } else {
      // Payment method creation failed, handle error
    }
  }

  @override
  Widget build(BuildContext context) {
    return Column(
      children: [
        ElevatedButton(
          onPressed: _isProcessingPayment ? null : _handlePayment,
          child: Text(_isProcessingPayment ? 'Processing...' : 'Pay Now'),
        ),
        SizedBox(height: 16),
        Text(
          'Estimated Cost: \$${widget.estimatedCost}',
          style: TextStyle(fontSize: 18),
        ),
        SizedBox(height: 8),
        Text(
          'Final Amount: \$${_finalAmount.toStringAsFixed(2)}',
          style: TextStyle(fontSize: 18),
        ),
        SizedBox(height: 16),
        StripePayment.addSource(
          source: PaymentSource.custom(
            name: 'Tip',
            amount: 0,
            currency: 'USD',
          ),
          onSourceAdded: (source) {
            StripePayment.confirmPaymentIntent(PaymentIntent(
              paymentMethodId: '',
              currency: 'USD',
              amount: (source.amount! * 100).toInt(),
              confirmationMethod: PaymentIntentConfirmationMethod.manual,
              confirm: true,
            ));
          },
        ),
      ],
    );
  }
}
```

This widget accepts an estimatedCost value and an onPaymentSuccess callback function as parameters. The estimatedCost is used to calculate the final amount to be charged (which is 20% more than the estimated cost), and the onPaymentSuccess callback function is called when the payment is successfully completed.

The widget contains an "Pay Now" button that triggers the payment flow when pressed. When the payment is completed, the final amount is displayed and the onPaymentSuccess callback function is called with the final amount as a parameter.

Additionally, the widget also includes a Stripe Payment Source that allows users to add a tip to their payment. When the "Tip" option is selected, a new PaymentIntent is created with a 0 amount and the user is prompted to confirm the payment.

You can add this custom widget to your FlutterFlow app by creating a new widget, copying the code above, and then using the widget in your app wherever payment functionality is needed. Be sure to replace 'YOUR_PUBLISHABLE_KEY_HERE' with your actual Stripe publishable key.
