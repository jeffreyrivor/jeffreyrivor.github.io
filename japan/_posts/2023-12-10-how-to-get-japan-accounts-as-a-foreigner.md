---
date: 2023-12-10 18:15:00 -0800
modified_date: 2025-05-26 12:45:00 -0700
---
# How to get Japan accounts as a foreigner

As a frequent traveler to Japan, a lot of the experiences or products I'm interested in are often tied
to online accounts that are only practical to obtain if you are a Japanese resident. Here are some
methods that I've used in the last 2 years to get over these hurdles even as a foreign visitor.

## Hurdle 1: Japanese cell phone number

A lot of account signups meant only for Japanese locals require a Japanese phone number that can send
and receive SMS and voice calls. Having a cell phone is pretty ubiquitous, smart or "dumb" phone.

The method I used was to get a [Mobal Long Term Japan SIM Card](https://www.mobal.com/japan-sim-card/)
which gives you a real Softbank Japanese phone number while in Japan with a monthly pool of 4G data and
unlimited "slow" data speeds after the pool is exhausted. It is important to get the **long term** plan
so you can keep the phone number indefinitely (even while not in Japan) by keeping the account current
(paying the monthly rate). If a short term time-limited plan is used, the phone number will be lost and
any account tied to the phone number may break in the future due to losing the ability to receive SMS
and calls, or due to the number being reissued to someone else that then uses it to unintentionally
"take over" the accounts associated with the number when they sign up for the same services.

## Hurdle 2: Japanese-issued payment method (prepaid credit card)

Foreigners will have a particularly hard time making online purchases because it is impossible to get
a Japanese bank account for them, and sometimes online services use a payment processor that doesn't
support foreign credit cards. When submitting a credit card for payment, "Japan-issued" is often not
listed as a requirement and will simply fail without a clear reason; the presence of an international
credit card brand in the list does not guarantee your foreign credit card will work.

~~The method I used was to get a **Japanese** [LINE Pay](https://pay.line.me/portal/global/about/sign-up)
account using the Japanese SIM I obtained from Mobal (note: signing up for LINE initially will require
receiving an SMS code on the Japanese cell phone number you sign up with). This is a prepaid virtual
credit card issued in Japan on the Visa payment network, so any site that requires a Japanese-issued
Visa card should work. However, charging the card requires a Japanese bank account, 7-11 ATM, or Lawson
"Loppi" copy machines. This effectively means you have to plan your purchases accordingly and load the
card while in Japan.~~ As of April 30, 2025, LINE Pay service in Japan
[has been discontinued](https://www.lycorp.co.jp/en/news/release/008632/). Full feature deprecation
timeline is available in Japanese at <https://line-pay-info.landpress.line.me/payment-info/>.

The method I used was [Vandle Card](https://vandle.jp/) using the Japanese SIM I obtained from Mobal
(note: signing up for Vandle initially will require receiving an SMS code on the Japanese phone number
you sign up with). However, the Android app is only available in the Japanese Google Play store so a
Japanese Google account will be required (multiple Google accounts can be signed into Google Play
simultaneously on Android phones); I have not tested with the Apple App Store. Unlike LINE Pay, there
are methods to load the prepaid balance that can be completed online (in addition to 7-11 ATM, Lawson
ATM, and other convenience store methods), namely credit cards that support 3D Secure (which I have
tested successfully using an American Express issued by American Express National Bank in the U.S.).

### Payment examples

For one broken example, the [au PAY mobile app](https://aupay.auone.jp/#appli) is a QR code payment
system that uses a prepaid digital wallet charged using a Japanese bank account, 7-11 Japan ATM, certain
Japanese-issued Visa-branded credit cards, JCB-issued credit cards, and "all" Mastercard, American Express,
and Diner Club International credit cards. However, attempting to input an American Express issued by
American Express National Bank in the U.S. fails, so I was left with no better charging methods compared
to the LINE Pay prepaid Visa.

One example that somehow works is [Lawson Ticket](https://l-tike.com/). Whether it's an event- or
venue-specific subsite for overseas sales or the Japan-only main site, using an American Express issued
in the U.S. (the same one I tried on au PAY) succeeds. Other sites using Lawson Ticket's account system
probably also work; I've successfully used [HMV & Books Online](https://www.hmv.co.jp/) with a U.S.
shipping address (side note: while the shipping costs can be steep because of using international air
freight, they are meticulous in packaging everything to prevent damage in transit).

Another example that was previously broken and now works as of March 2025 is [e+](https://eplus.jp) with
American Express issued by American Express National Bank in the U.S. Note that the "overseas" e+ site
is separate and has generally broader support for credit cards, whereas the Japanese site requires
receiving an SMS code on the Japanese cell phone number you sign up with in addition to generally
supporting only Japanese-issued credit cards until the American Express started working.

## Situational: geo-blocking

Some sites use your public IP address to impose geo-restrictions to only allow Japan-based users. One
specific example I've encountered is the **Japanese** sections of Lawson Ticket (the events or venues
only allowing Japanese to purchase); it manifests as a generic login failure. For this, using a VPN
like [ExpressVPN](https://www.expressvpn.com/) with a Japan location has worked for me.
