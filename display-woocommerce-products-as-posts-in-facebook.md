# Integrating Facebook and WooCommerce : Publish Woocommerce Product in Facebook Page as post

This guide outlines the step-by-step process of integrating Facebook and WooCommerce for seamless management of your online store and social media presence.

## Setting Up Facebook Integration

1. **Create a Facebook App**:
   - Go to the [Facebook for Developers](https://developers.facebook.com/) portal.
   - Click on "My Apps" and then "Create App".
   - Choose the appropriate app type (e.g., Business).
   - Follow the on-screen instructions to complete the app creation process.

2. **Retrieve App ID and App Secret**:
   - Once your app is created, navigate to its dashboard.
   - You'll find your App ID and App Secret there. Keep these credentials secure as they are required for accessing Facebook's APIs.

3. **Set Up Facebook Page**:
   - If you haven't already, create a Facebook Page for your business or organization.
   - Ensure that you have admin access to this page as you'll need it for posting content programmatically.

4. **Generate User Access Token**:
   - Obtain a User Access Token with the necessary permissions for managing your Facebook page.
   - You can generate this token manually through Facebook's Graph API Explorer or via a server-side process.

## Integrating WooCommerce

1. **Install WooCommerce**:
   - If you haven't already, install and set up WooCommerce on your WordPress website.
   - WooCommerce is a plugin that adds e-commerce functionality to your WordPress site.

2. **Set Up WooCommerce REST API**:
   - Enable the WooCommerce REST API by navigating to WooCommerce > Settings > Advanced > REST API.
   - Generate API keys by going to WooCommerce > Settings > Advanced > REST API > Keys/Apps and clicking on "Add Key".
   - Create a key by specifying a description, user, and permissions (Read/Write) and generate the keys.

3. **Retrieve WooCommerce Store URL**:
   - Note down the base URL of your WooCommerce store. It usually follows the format `https://yourdomain.com`.

4. **Test WooCommerce API**:
   - Before integrating with other systems, ensure that your WooCommerce API is working correctly.
   - You can test it using tools like Postman or cURL by making sample requests to your store's API endpoints.

## Integrating Facebook and WooCommerce

1. **Install Required Libraries**:
   - Use Composer to install the necessary PHP libraries for integrating with Facebook and WooCommerce.
   - Run `composer require facebook/graph-sdk automattic/woocommerce` in your project directory.

2. **Configure the Script**:
   - Copy the provided PHP script into your project directory.
   - Update the script with your Facebook App credentials (App ID, App Secret), Facebook page ID, User Access Token, and WooCommerce API credentials (Store URL, Consumer Key, Consumer Secret).

3. **Execute the Script**:
   - Run the PHP script from the command line using `php your_script_name.php`.
   - Ensure that the script executes without errors and successfully fetches products from WooCommerce and posts them on Facebook.

4. **Test and Debug**:
   - Test the integration thoroughly in a development environment.
   - Monitor for any errors or issues during product fetching or posting.
   - Debug as necessary and make adjustments to the script for optimal performance.

5. **Schedule and Automate**:
   - Once the integration is working as expected, consider scheduling the script to run at regular intervals using cron jobs or task schedulers.
   - This allows you to automate the process of updating your Facebook page with new products from your WooCommerce store.

By following these steps, you can effectively integrate Facebook and WooCommerce to streamline your e-commerce operations and enhance your online presence.

```php
// Including required libraries
use Facebook\Facebook;
use Automattic\WooCommerce\Client;

// Include Composer's autoloader
require_once 'vendor/autoload.php';

// Facebook App credentials
$appId = 'YOUR_APP_ID';
$appSecret = 'YOUR_APP_SECRET';

// Facebook page ID
$pageId = 'YOUR_PAGE_ID';

// User Access Token for posting on Facebook
$userAccessToken = 'YOUR_USER_ACCESS_TOKEN';

// Initialize Facebook SDK
$fb = new Facebook([
    'app_id' => $appId,
    'app_secret' => $appSecret,
    'default_graph_version' => 'v2.5'
]);

// Get long-lived access token from user access token
$longLivedToken = $fb->getOAuth2Client()->getLongLivedAccessToken($userAccessToken);
$fb->setDefaultAccessToken($longLivedToken);

// Fetch forever page access token
$response = $fb->sendRequest('GET', $pageId, ['fields' => 'access_token'])->getDecodedBody();
$foreverPageAccessToken = $response['access_token'];

// Initialize WooCommerce client
$woocommerce = new Client(
    'YOUR-SITE-URL',
    'CONSUMER-KEY',
    'SECRET-KEY',
    [
        'wp_api' => true,
        'version' => 'wc/v3',
        'verify_ssl' => false,
    ]
);

try {
    // Fetch products from WooCommerce
    $products = $woocommerce->get('products', ['per_page' => 10]);
    foreach ($products as $product) {
        // Extract product details
        $productName = $product->name;
        $productShortDescription = $product->short_description;

        try {
            // Post product on Facebook
            $fb->setDefaultAccessToken($foreverPageAccessToken);
            $fb->sendRequest('POST', "$pageId/feed", [
                'message' => $productName,
                // 'link' => $productImageUrl,
            ]);
        } catch (Facebook\Exceptions\FacebookResponseException $e) {
            echo 'Graph returned an error: ' . $e->getMessage();
        } catch (Facebook\Exceptions\FacebookSDKException $e) {
            echo 'Facebook SDK returned an error: ' . $e->getMessage();
        }
    }
} catch (Exception $e) {
    echo 'Error Fetching Products' . $e->getMessage();
}

```
## Code Explanation:

1. **Facebook SDK Initialization**:
   - The Facebook SDK is initialized with the provided App ID, App Secret, and default graph version.

2. **Fetching Long-lived Access Token**:
   - The script retrieves a long-lived access token using the provided user access token. This is required for extended usage.

3. **Fetching Forever Page Access Token**:
   - The script retrieves a permanent (forever) page access token for posting on the specified Facebook page.

4. **WooCommerce Client Initialization**:
   - The WooCommerce client is initialized with the store URL, consumer key, consumer secret, and other options like API version and SSL verification.

5. **Fetching Products from WooCommerce**:
   - Products are fetched from the WooCommerce store using the `get` method. It retrieves a specified number of products per page.

6. **Posting Products on Facebook**:
   - For each product fetched, the script attempts to post the product name as a message on the specified Facebook page.

7. **Error Handling**:
   - Exceptions are caught and appropriate error messages are displayed if there are any issues during product fetching or posting.

This script automates the process of fetching products from a WooCommerce store and posting them on a Facebook page. It's essential to ensure that all credentials are correctly configured and permissions are granted for smooth execution.
