# Aegis client demo and pitch

## What to say in one sentence

"Aegis helps someone see a short-term loan's full cost and whether repayment collides with their paydays and bills, then pauses the decision when their own data shows a pressure point."

## Set up the local demo

From this project directory:

```sh
npm run demo:csv
npm start
```

Use the local URL printed by the app. The sample CSV at `examples/shield-demo-transactions.csv` contains invented transactions dated relative to the day you run the command. Member sign-in details for the fictional local account are in `LOCAL-ACCESS.local.md`, which is private and excluded from Git. If the app is already running, keep using that local URL.

## Two-minute no-login walkthrough

1. Open `/check` and say: "The consumer starts with the loan offer, not a credit score."
2. Click **Use a fictional sample instead**. Point out the $500 received, $575 repaid, $75 cost, the 30-day shortfall, and the **Aegis Shield** review. The site pauses its continue actions until the warning is acknowledged.
3. Click **Edit the offer**, expand **Check the offer's wording**, and paste: `Guaranteed loan. Pay an upfront fee before funds are released. Act now.` Click **Use a fictional sample instead** again. Show the highlighted phrase, FTC source, and the extra pause. Explain that this is a warning to verify the offer, not a declaration that a named lender is fraudulent.
4. Open the expandable research context. Show that aggregate CFPB complaint findings inform the review questions while the consumer's own numbers determine the cash-flow result.

## Member walkthrough with fictional data

1. Sign in through `/login` using the fictional member credentials in `LOCAL-ACCESS.local.md`.
2. On **Privacy**, turn on **Upload transaction history from a CSV file**, then **Turn on my personal transaction shield**. Turn on **Use details I enter to compare this offer** and **Save my offer comparisons** only if you want to demonstrate the save pause.
3. On **My Shield**, upload `examples/shield-demo-transactions.csv`. Aegis immediately refreshes the analysis and shows possible repeated short-term credit, repeated fees, and variable income-like deposit descriptions. It also shows the number of repeating-item suggestions, which the member must confirm before adding to a forecast.
4. On **Check offer**, enter a $0 starting balance, $500 received today, and one $575 payment 14 days later. Click **See my result**. Aegis shows a projected shortfall. Click **Save this comparison** to demonstrate the pause before saving; choose **Change the offer details** to leave the comparison unsaved.
5. Show the optional virtual bills-reserve preview after adding expected income and bills. It recalculates the suggested allocation locally; it does not transfer money.

The CSV is a disposable fictional example. If the same file was already imported into the demo account, the app will reject a duplicate. Use the existing imported patterns or regenerate the file on another date.

## 45-second pitch

"Short-term loans can look manageable when someone sees only the cash arriving today. The difficult part is the repayment date: it can collide with rent, essential bills, or a low-income week. Aegis puts the total repayment and a 30-day cash-flow comparison in front of the consumer before they decide. With permission, it checks their imported transaction descriptions for repeat borrowing, fees, and variable income-like deposits. It pauses on a projected shortfall or concerning offer wording and explains the evidence. Public CFPB complaint data focuses the review questions. The consumer controls the data and the decision. A credit union could offer this as a member benefit and evaluate whether the warnings help members avoid costly repeat borrowing."

## Answer likely client questions honestly

- **Is this connected to banks?** No. Members can import their own CSV or enter amounts and dates. There are no bank passwords or live account feeds.
- **Does it move money or stop a lender's checkout?** No. Bill routing is a preview, and pauses happen inside Aegis. Real transfers or external checkout interception require partner integrations and separate authorization.
- **Does AI diagnose vulnerability or decide that a lender is predatory?** No. It uses transparent transaction and offer cues, public-data research context, and the person's cash-flow arithmetic. Public loan outcomes are not a validated personal scam model.
- **Is it ready for public deployment?** No. This is a working local pilot build. A real rollout requires security review, staff MFA, bank-data and payment partners if those features are wanted, and evaluation with consenting users.

## Proposed pilot success measures

Ask members, with separate consent, whether warnings were understandable and whether they changed, delayed, or declined an offer. Measure false alerts, completed reviews, and repeat use. Do not claim poverty reduction or savings until a pilot measures them.
