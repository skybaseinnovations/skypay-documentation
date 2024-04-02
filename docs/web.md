# Web Integration

### **How Skypay Makes Payments Easy:**

1. **Start Shopping**: A customer picks out something nice on your website and decides to buy it. As they do, your system creates a special code called a **`code`** that will be reference for the payment for their order.
2. **Making the Payment**: With their shopping cart full, the customer hits the checkout button. This sends their order details, including the **`code`**, the total cost, and a **`return_url`** (so they can easily get back to your site), over to Skypay.
3. **Payment Time**: The customer is then taken to the Skypay payment page (**`https://pay.skypay.com`**), where they pay for their purchase throught various options, as setup on your merchant dashboard.
4. **Success and Back**: After the payment is done, Skypay lets your site know everything went well by sending a message to the **`return_url`** you provided.
5. **Check It Twice**: If you want, you can double-check to make sure the payment was received. This can be done using the payment-verify API
6. **All Done**: Now that the payment is confirmed, you can wrap up the order. This might mean sending a product, providing a service, or giving the customer access to something special online.
7. **Happy Customer**: The customer is happy with their smooth shopping experience, and you've successfully made a sale. Everyone wins.


## **Generating Checkout Link**

#### Environment 

=== "UAT (Testing)"

    ``` 
    <base-app-url>  = https://app-uat.skypay.dev
    ```

=== "Live"

    ```
    <base-app-url>  = https://app.skypay.dev
    ```


### **Method 1: Using Direct URL**
You can construct a direct URL to the Skypay checkout page by concatenating your parameters into a query string. This method is particularly useful for scenarios where you want to quickly redirect users to the payment process without a server-side action or when directly linking from an email, SMS, or within your application.

#### **Direct URL Structure**

```
<base-app-url>/checkout?api_key=<API_KEY>&amount=<AMOUNT>&success_url=<SUCCESS_URL>&failure_url=<FAILURE_URL>&code=<UNIQUE_ORDER_ID>

```

Simply replace **`<API_KEY>`**, **`<AMOUNT>`**, **`<SUCCESS_URL>`**, **`<UNIQUE_ORDER_ID>`** and **`<FAILURE_URL>`** with your actual data. This URL can then be used in hyperlinks or redirects to initiate the payment process directly.

#### **Example Usage in HTML**

```html
<a href="<base-app-url>/checkout?api_key=453440321&amount=100&success_url=https://example.com/success&failure_url=https://example.com/failure&code=2093" target="_blank">Pay with Skypay</a>
```

This creates an anchor tag that users can click to be directly taken to the Skypay checkout page with the specified parameters.

### **Method 2: Through a GET Form**

Alternatively, you can use an HTML form to submit these parameters via a GET request. This method provides a seamless user experience for websites, allowing for a more integrated look and feel.

#### **HTML Form Structure**

```html
<form action="<base-app-url>/checkout" method="GET">
  <input type="hidden" name="api_key" value="453440321">
  <input type="hidden" name="amount" value="500">
  <input type="hidden" name="code" value="993838">
  <input type="hidden" name="success_url" value="https://example.com/success">
  <input type="hidden" name="failure_url" value="https://example.com/failure">
  <input type="submit" value="Pay with Skypay">
</form>
```

In this form, the **`action`** attribute points to the Skypay checkout URL, and method **`GET`** is specified to append form data into the URL as query parameters. Users will click the "Pay with Skypay" button to initiate the payment process, leading them to the Skypay checkout page.

### **Parameters**

| Parameter | Description |
| --- | --- |
| api_key | The unique api key for the merchant, available in your dashboard |
| amount | The transaction amount |
| success_url | URL where users are redirected upon success. |
| failure_url | URL where users are redirected upon failure. |
| code | Unique identifier for the order, usually order’s id, this will be required later to verify |

### **Handling Redirects & Callbacks**

#### Success:

When a transaction is completed successfully, users are automatically redirected to the **`success_url`** specified in your payment form. Along with the redirection, an additional query parameter **`?data=<base_64_encoded_data>`** is appended to the URL. This parameter contains base64 encoded JSON data which, when decoded, can offer valuable information for tracking the transaction or updating order status on your system.

#### Failure:

Should a transaction not go through, users are redirected to the **`failure_url`** you've set up. To help diagnose or inform users of the issue, a query parameter is included in the format **`?message=error_description`**. This parameter provides a brief explanation or code indicating why the transaction failed, aiding in troubleshooting or customer support efforts.

### **Additional Notes**

- **Decoding the `data` Parameter**: Upon successful transaction redirection, it's recommended to decode the base64 data server-side to maintain security. This decoded JSON can include transaction details such as **`code`**, amount, and transaction status.
- **Handling Failure Messages**: The **`message`** parameter on failure should be properly handled to display user-friendly messages or logs for internal tracking. Be cautious not to expose sensitive or technical details directly to users to avoid confusion or security concerns.

### **Verify Payment**

Skypay offers a robust API endpoint for merchants to verify the status of a payment transaction. This verification step is crucial for confirming that a payment has been successfully processed before finalizing the order on the merchant's side. Below is a detailed guide on how to use the Verify Payment API.

#### Endpoint
**Method: `GET`**

=== "UAT (Testing)"

    ``` 
    https://api-uat.skypay.dev/api/v1/checkout/payments-verify/{code_or_order_id}
    ```

=== "Live"

    ```
    `GET` https://api.skypay.dev/api/v1/checkout/payments-verify/{code_or_order_id}
    ```

Replace **`{code_or_order_id}`** with the payment code or your order ID provided at the time of transaction initiation.

**Additional Headers:**
    
    Key: <API-KEY from Merchant Dashboard>

**Example Request**

    let config = {
    method: 'get',
    url: 'https://api-uat.skypay.dev/api/v1/checkout/payments-verify/<your_order_id>',
    headers: { 
        'Key': '<your_api_key>', 
    }
    };
    
Replace <your_order_id> with your actual order ID and <your_api_key> with your Skypay API key.

### Successful Response

A successful request to the Verify Payment API will return a JSON object containing details about the payment transaction. Below is an example of a successful response:

```json

{
    "status": true,
    "message": "Existing payment found",
    "data": {
        "id": "23dfbfb4-8476-41f0-95b2-38bf79affd2a",
        "user_provider_id": "56bc5495-1b49-45f8-a37b-aef2ce104216",
        "provider_id": "001",
        "provider_name": "eSewa",
        "provider_code": "esewa",
        "provider_logo_url": "https://esewa.com.np/common/images/esewa_logo.png",
        "code": "46895",
        "mode": "API",
        "created_at": "2024-04-01T11:44:05.000000Z",
        "expires_at": "2024-04-01T17:33:18.000000Z",
        "status": "complete",
        "amount": 100,
        "payment_data": {
            "transaction_code": "0007680",
            "status": "COMPLETE",
            "total_amount": "100.0",
            "transaction_uuid": "7811574",
            "product_code": "EPAYTEST",
            "signed_field_names": "transaction_code,status,total_amount,transaction_uuid,product_code,signed_field_names",
            "signature": "VtscWlbCOHfkEE1aSdNckBf4nKyiwLa/Da15FtNu8/M="
        }
    }
}

```

***Understanding the `status` and `payment_data` in the Verify Payment API Response**

#### Payment Status (**`status`**):

The **`status`** field within the **`data`** node of the API response indicates the current status of the transaction. It's crucial for determining the outcome of a payment request. Here are the possible values for **`status`** and their meanings:

- **`complete`**: The transaction was successful. You should proceed with fulfilling the order.
- **`pending`**: The transaction is still in process. You may need to check again later.
- **`invalid`**: The transaction request contained errors or was not processed due to invalid parameters. It is recommended to review the request and possibly prompt the user to retry.
- **`cancelled`**: The transaction was cancelled by the user or due to timeout. You may need to prompt the user to initiate a new payment request if they wish to proceed with the purchase.

For successful order processing and fulfillment, ensure that the **`status`** is **`complete`**. This verification is critical to confirm that the payment has been securely processed and received.

#### Payment Provider Response (**`payment_data`**):

The **`payment_data`** node contains detailed information from the payment provider regarding the transaction. Since Skypay integrates with multiple payment gateways, the structure and content of **`payment_data`** may vary depending on the provider. Please refer the respective documentation of the payment provider in case of any confusions.

### Error Handling

In case of a failed transaction or if the payment cannot be found, the API will return a relevant error message and status. Ensure your application is equipped to handle such responses gracefully, informing the user or taking appropriate automated action.

### Notes

- It's critical to perform this verification step before proceeding with order fulfillment to ensure the integrity of the transaction.
- The **`status`** field within the **`data`** object indicates the payment status, which can be used to automate next steps in your order processing workflow.
