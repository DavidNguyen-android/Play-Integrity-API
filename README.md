# Google Play Integrity API

## Overview

This document provides a comprehensive description of the Google Play Integrity API and Firebase App Check using Play Integrity for enhancing app security.

## Google Play Integrity API

### Introduction
The Google Play Integrity API helps protect your app by verifying the integrity of the app, the device, and the user actions. It helps ensure that your app is running in a secure environment and has not been tampered with.

### Features
- **App Integrity**: Ensures that the app is the genuine version distributed by Google Play.
- **Device Integrity**: Confirms that the app is running on a genuine, unmodified device with Google Play services.
- **Account Integrity**: Validates the user's account, ensuring it has not been compromised.

### Standard vs. Classic API Requests

##### Standard API request for Play Integrity

This document explains how to request an integrity verdict using the Play Integrity API's Standard API. 

###### Process Overview

1. **Preparation:**
    *  Enable Play Integrity in your Play Console [https://developer.android.com/google/play/integrity](https://developer.android.com/google/play/integrity).
    *  Implement an `StandardIntegrityTokenProvider` in your app.
2. **Requesting a verdict:**
    *  Generate a `requestHash` of the user action you want to protect.
    *  Construct a `StandardIntegrityTokenRequest` object with the `requestHash`.
    *  Use the `StandardIntegrityTokenProvider` to call `request()` with the `StandardIntegrityTokenRequest`.
3. **Decrypting and verifying the verdict:**
    *  The Play Integrity API responds with an encrypted token.
    *  Set up a service account linked to your app's Google Cloud project.
    *  Use the Google API Client Library to decrypt the token on Google's servers and obtain the verdict.

###### Key Points

*  **Security:** The `requestHash` protects your request from tampering.
*  **Decryption:**  You need a service account to decrypt the response token on Google's servers.
*  **Client Libraries:** Use the Google API Client Library to simplify API calls.

##### Classic API request for Play Integrity

This markdown explains how to use the Classic API for requesting an integrity verdict from the Play Integrity API.

######  Process Flow

1. **Preparation:**
    *  Enable Play Integrity in your Play Console [https://developer.android.com/google/play/integrity](https://developer.android.com/google/play/integrity).
    *  Generate a nonce (a random value).
2. **Constructing the Request:**
    *  Create an `IntegrityManager` object.
    *  Build an `IntegrityTokenRequest` using the `setNonce()` method and provide the generated nonce.
    *  If your app is distributed outside Google Play or is an SDK, include your Google Cloud project number using `setCloudProjectNumber()`. Apps on Google Play don't require this step.
3. **Sending the Request:**
    *  Use the `IntegrityManager` to call a method for sending the request (implementation varies by platform).

###### Response and Verification

1. **Encrypted Verdict:**
    *  The Play Integrity API responds with an encrypted token containing the integrity verdict.
2. **Decryption Options:**
    * **Client-side decryption (not recommended):**
        *  Set up a service account for your app's Google Cloud project.
        *  Use the Google API Client Library to decrypt the token locally. This approach requires additional security measures on your end.
    * **Server-side decryption (recommended):**
        *  Send the encrypted token to your server.
        *  On the server, use the Google API Client Library to decrypt the token on Google's servers for enhanced security.

##### Compares
This markdown table compares Standard and Classic API requests, highlighting their pros and cons:

| Feature | Standard API Request | Classic API Request |
|---|---|---|
| **Latency** | Lower (few hundred milliseconds) | Higher (few seconds) |
| **Reliability** | High | Moderate |
| **Data Usage** | Lower | Higher |
| **Battery Consumption** | Lower | Higher |
| **Security** | More secure | Less secure (requires additional mitigation) |
| **Use Cases** | Frequent checks, on-demand requests | Infrequent checks, highly sensitive actions |
| **Caching** | Not recommended | Not recommended (use standard requests instead) |

[Docs Standard API Requests](https://developer.android.com/google/play/integrity/standard)
[Docs Classic API Requests](https://developer.android.com/google/play/integrity/classic)
[Docs Standard vs. Classic API Requests](https://developer.android.com/google/play/integrity/classic#compare-standard)

**Pros of Standard API Requests:**

* **Faster response times:** Ideal for frequent checks and real-time applications.
* **Highly reliable:** Provides consistent and dependable results.
* **Lower resource consumption:** Reduces data usage and battery drain on devices.
* **Enhanced security:** Mitigates risks associated with certain attacks.

**Cons of Standard API Requests:**

* **May not be suitable for all scenarios:** Might not be ideal for infrequent checks of highly sensitive actions.

**Pros of Classic API Requests:**

* **Legacy support:** Useful for applications built for older API versions.

**Cons of Classic API Requests:**

* **Slower response times:** Less suitable for real-time applications.
* **Lower reliability:** Higher chance of encountering issues.
* **Higher resource consumption:** Increases data usage and battery drain.
* **Reduced security:** Requires additional security measures to mitigate risks.

### Implementation

#### Requirements
- Link your app product with Google Cloud.
- Configure Google Cloud Platform (GCP) to get authentication keys for the backend.

#### Steps
1. **Add Google Play Integrity API**:
    - Add the required dependencies to your project.
    ```gradle
    implementation 'com.google.android.play:integrity:1.3.0'
    ```

2. **Obtain Integrity Token**:
    - Call the Integrity API to obtain a token.
    ```kotlin
    private fun getIntegrityToken() {
        val nonce: String = generateNonce()
        // Create an instance of a manager.
        val integrityManager = IntegrityManagerFactory.create(applicationContext)
        // Request the integrity token by providing a nonce.
        val integrityTokenResponseTask = integrityManager.requestIntegrityToken(
            IntegrityTokenRequest.builder()
                .setCloudProjectNumber(345641564)// CloudProjectNumber form google cloud
                .setNonce(nonce)
                .build()
        )
        integrityTokenResponseTask.addOnSuccessListener { integrityTokenResponse: IntegrityTokenResponse ->
            val integrityToken = integrityTokenResponse.token()
            //In here you should send integrityToken to your app's backend
            sendIntegrityTokenToServer(integrityToken)
        }
        integrityTokenResponseTask.addOnFailureListener { e: Exception? ->
            Log.e("IntegrityToken error", e?.message.toString())
        }
    }
    
    private fun generateNonce(): String {
        val length = 50
        var nonce = ""
        //In here you get unique value form your server
        val allowed = "test_test_test"
        for (i in 0 until length) {
            nonce += allowed[floor(Math.random() * allowed.length).toInt()].toString()
        }
        return nonce
    }
    ```

3. **Verify Token on Backend**:
    - Send the token to your backend server.
    - Use Google APIs to verify the token and extract integrity verdicts.
    ```javascript
    const { GoogleAuth } = require('google-auth-library');
    const auth = new GoogleAuth();
    const client = await auth.getIdTokenClient('YOUR_BACKEND_API_URL');

    // Send the token to the Google Play Integrity API for verification
    const response = await client.request({
        url: 'https://playintegrity.googleapis.com/v1/{token}:decodeIntegrityToken',
        method: 'POST',
        data: {
            integrityToken: token
        }
    });

    const integrityVerdict = response.data;
    ```

#### Integrity Verdicts
- **requestDetails**: Validates the request from the app.
- **appIntegrity**: Confirms the app is installed from Google Play.
- **deviceIntegrity**: Checks if the app runs on a genuine device with Google Play services.
- **accountDetails**: Validates the user's account status.
- **environmentDetails**: Confirms the device environment and security status.

[Docs Integrity Verdicts](https://developer.android.com/google/play/integrity/verdicts)

### Notes
- **Limitations**: The default request limit is 10,000 per day. Requests to increase this limit can take up to 7 days.
- **Error Handling**: Ensure proper error handling for token requests and decryption.

**Recommendation:**

For most cases, Standard API requests are the preferred choice due to their faster speed, reliability, and lower resource consumption. Classic API requests should be reserved for infrequent checks of highly sensitive actions, especially in legacy applications.


