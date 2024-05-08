# Purchase
- Double spending
  - Purchase a product and save the request that have **Resnum** and **RefNum**.
  - Create a new purchase, but don't pay for it.
  - Use the **ResNum** of new purchase request (that is sending to PSP), in the previuos purchase request for double spending attack.
- Using a paid feature
  - Remove the `link` parameter in update request of Divar advertisement, to show link in the advertisement, without pay for it.
- Reduce the price
  - Change the value of cost (like `price=`) in the request of add to market backet.
  - Change the quantity of products to a negative value (like `-5`) and check if it affects the final price of market basket.
  - Use one coupon to get unlimited discount.
  - Use two coupons one after another again and again.
  - Increase the number of product until price overflow and make the price negative, then calculate the needed number of requests to make the price under 10 dollars.
    - Burp Intruder: Paylods
      - Payload Type: `Null Paylods`
      - Payload Options: Use `Continue indefinetly` for detection and `Generate X payloads` for accurate number of request.
    - Burp Intruder: Resource Pool
      - Maximum concurrent requests: `1`
- Bypass purchasing workflow
  - Buy a product and check redirecting endpoint if there is an order confirmation page. (like `GET /cart/order-confirmation?order-confirmation=true`)
  - Then intercept the buying request of other product and change the redirection endpoint to above one.


# Access Control
- OTP
  - Required parameter from victim + OTP of attacker  →  login with victim user
- Change other users passwords
  - In state-changing endpoints (like change password), remove `current-password` from request, to bypass checking of current password.
- Intercepts requests that are sent after login and drop if they are defining access levels.(like `GET /role-selector`).


# Resource
- https://book.hacktricks.xyz/pentesting-web/bypass-payment-process
