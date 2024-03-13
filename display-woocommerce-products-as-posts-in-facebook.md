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

Wordpress Page Template
App Type: Other > Buisness

```php
/**
 * Template Name: Social-feed
 **/
require_once get_template_directory() . '/vendor/autoload.php';
use Facebook\Facebook;
use Automattic\WooCommerce\Client;
$site_url = 'Your-site-url';
$consumer_key = 'Your-consumerKey';
$secret_key = 'Your-secret-key';

$appId = 'App-id';
$appSecret = 'Your-secret-id';
$pageId = 'Page-id';
$AccessToken = 'Your-Access Token';

// Function to retrieve stored long-lived access token from options

function get_stored_facebook_token() {
    return get_option('facebook_long_lived_token');
}

function is_valid_facebook_token($token) {
    $storedTime = get_option('facebook_token_stored_time');
    $expiryTime = strtotime($storedTime . ' +60 days');
    if (time() < $expiryTime) {
        return true; // Token is still valid
    } else {
        return false; // Token has expired or is invalid
    }
}

// Retrieve stored long-lived access token from options table
$storedToken = get_stored_facebook_token();
// Check if stored token is valid
if (is_valid_facebook_token($storedToken)) {
    echo "Stored token is valid.<br>";
    $fb = new Facebook([
        'app_id' => $appId,
        'app_secret' => $appSecret,
        'default_graph_version' => 'v2.5'
    ]);
    $longLivedToken = $storedToken; // Reuse stored token

    $fb->setDefaultAccessToken($storedToken);

} else {

    try {
        // Token is stored, generate a longlived token
        $fb = new Facebook([
            'app_id' => $appId,
            'app_secret' => $appSecret,
            'default_graph_version' => 'v2.5'
        ]);
        
        // Generate new long-lived access token
        $longLivedToken = $fb->getOAuth2Client()->getLongLivedAccessToken($AccessToken);
        
        // Store the new long-lived access token
        update_option('facebook_long_lived_token', $longLivedToken);
        update_option('facebook_token_stored_time', current_time('mysql'));
        
        // Set the new long-lived access token as default
        $fb->setDefaultAccessToken($longLivedToken);
        
        echo "New long-lived token generated and stored.<br>";
    } catch (Facebook\Exceptions\FacebookResponseException $e) {
        echo 'Graph returned an error: ' . $e->getMessage();
    } catch (Facebook\Exceptions\FacebookSDKException $e) {
        echo 'Facebook SDK returned an error: ' . $e->getMessage();
    } catch (Exception $e) {
        echo 'Error: ' . $e->getMessage();
    }
    
}

$response = $fb->sendRequest('GET', $pageId, ['fields' => 'access_token'])->getDecodedBody();

$foreverPageAccessToken = $response['access_token'];
//Another way login redirect and fetch access token issue with fb popup blank on redirect

  // Initialize WooCommerce client
  $woocommerce = new Client(
      $site_url,
      $consumer_key,
      $secret_key,
      [
          'version' => 'wc/v3',
      ]
  );
  
try {
   
    $allProducts = $woocommerce->get('products');
    $count = 0;
    // Loop through all products retrieved
    foreach ($allProducts as $product) {
        if($count === 2){
            break;
        }
        $productid = $product->id;        
        $productname = $product->name;
        $productShortDescription = $product->short_description;
        $productLink = $product->permalink;
        $productImageUrl = $product->images[0]->src; // Assuming the first image is the featured image
        $productPrice = $product->price;
        $alreadyPosted = get_post_meta($productid, 'posted', true);
        $fb->setDefaultAccessToken($foreverPageAccessToken);
        // Create post data if not already posted
        //avoid posting products duplication again and again when running code
        if ($alreadyPosted !== 'true') {      
            // Upload the image to the Facebook page's photos
            $response = $fb->sendRequest('POST', "/$pageId/photos", [
                'message' => $productname . "\nPrice: $" . $productPrice . "\n" . $productShortDescription ."\nProduct Link: " . $productLink,
                'url' => $productImageUrl,
                'caption' => $productLink, // Optional: Caption for the photo
                // 'no_story' => true
            ]);
            $imageId = $response->getGraphNode()->getField('id');

            update_post_meta($productid, 'posted', 'true');

            $graphNode = $response->getGraphNode();
            $postId = $graphNode['id'];

            echo 'Product posted successfully. FB Post ID: ' . $postId . ' Product-id ' . $productid . '<br>';
        };
        $count++;
    }
} catch (Facebook\Exceptions\FacebookResponseException $e) {
    echo 'Graph returned an error: ' . $e->getMessage();
} catch (Facebook\Exceptions\FacebookSDKException $e) {
    echo 'Facebook SDK returned an error: ' . $e->getMessage();
} catch (Exception $e) {
    echo 'Error: ' . $e->getMessage();
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

**How too add as Post feed**
```php
 try {
       $message = 'Test message website';
       $title = 'Post From Website';
       $description = 'Programming blog.';
       
       $attachment = array(
           'message' => $message,
           'name' => $title,
           'description' => $description,
          
       );
       // Post a message to the user's profile
       $response = $fb->post('/me/feed', $attachment ,$accessToken);

       // Get the post ID if the post was successful
       $graphNode = $response->getGraphNode();
       echo 'Post ID: ' . $graphNode['id'];
   } catch (FacebookResponseException $e) {
       echo 'Graph returned an error: ' . $e->getMessage();
   } catch (FacebookSDKException $e) {
       echo 'Facebook SDK returned an error: ' . $e->getMessage();
}
```

Generate access Token with Login Url
App Type : Facebook Login
configure Product: Facebook setup
Set up Redirect Url

Example shows how to fetch Access token With Login Url
```php

/**
 * Template Name: Social Feed
 **/

// Get the child theme directory
$theme_directory = get_stylesheet_directory();

// Include the autoloader provided in the SDK
require_once $theme_directory . '/templates/vendor/autoload.php';

use Facebook\Facebook;
use Facebook\Exceptions\FacebookResponseException;
use Facebook\Exceptions\FacebookSDKException;

// Set your Facebook App ID and App Secret
$appId = '882905023657603';
$appSecret = 'd85238e05bc730e561ed0af95d6b7e2a';
$redirectUri = 'https://foodsafetystandard.demo1.bytestechnolab.com/social-feed';
$scopes = ['publish_to_groups', 'pages_read_engagement', 'pages_manage_posts','email'];

$loginUrl = 'https://www.facebook.com/v12.0/dialog/oauth?' . http_build_query([
    'client_id' => $appId,
    'redirect_uri' => $redirectUri,
     'scope' => 'email',
    // 'scope' => implode(',', $scopes), 
    'response_type' => 'code',
]);

if (isset($_GET['code'])) {
    $tokenUrl = 'https://graph.facebook.com/v12.0/oauth/access_token';
    $postData = [
        'client_id' => $appId,
        'client_secret' => $appSecret,
        'redirect_uri' => $redirectUri,
        'code' => $_GET['code'],
    ];

    // Use PHP's built-in functions to send a POST request
    $options = [
        'http' => [
            'method' => 'POST',
            'header' => 'Content-type: application/x-www-form-urlencoded',
            'content' => http_build_query($postData),
        ],
    ];
    $context = stream_context_create($options);
    $response = file_get_contents($tokenUrl, false, $context);

    if ($response !== false) {
        $params = json_decode($response, true);
        if (isset($params['access_token'])) {
            $accessToken = $params['access_token'];
            echo 'Access Token: ' . $accessToken;
            //// Process Your code
        } else {
            echo 'Error: Access token not found';
        }
    } else {
        echo 'Error: Failed to get access token.';
    }
} else {
    header('Location: ' . $loginUrl);
    exit();
}
