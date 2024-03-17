
### Installing Google Library and Setting Up Project:

1. **Install Google API PHP Client Library**:
    - Use Composer to install the Google API PHP Client library. Run the following command in your project directory:
      ```bash
      composer require google/apiclient:^2.0
      ```
    - This will download and install the necessary files for interacting with Google APIs.

2. **Set Up Google Cloud Platform Project**:
    - Go to the [Google Cloud Console](https://console.cloud.google.com/).
    - Create a new project or select an existing one.
    - Navigate to the "APIs & Services" > "Library" section.
    - Enable the required APIs for your project, such as the Google Sign-In API.

3. **Create OAuth 2.0 Credentials**:
    - In the Cloud Console, navigate to the "APIs & Services" > "Credentials" section.
    - Click on "Create credentials" and select "OAuth client ID".
    - Choose the application type (e.g., Web application).
    - Add authorized redirect URIs where Google will redirect users after authentication. Ensure these URIs use HTTPS.
    - Click "Create" to generate client ID and client secret.

### Copy Client ID, Secret ID, Redirect URI:

1. **Copy Client ID and Secret ID**:
    - After creating OAuth 2.0 credentials, you'll receive a client ID and client secret.
    - Copy the client ID and client secret values provided by Google.

2. **Copy Redirect URI**:
    - Ensure to copy the redirect URI(s) you configured in the OAuth client ID setup.
    - These redirect URIs are where Google will redirect users after they authenticate. Remember that these URIs must use HTTPS for security reasons.

# Google Signup Credentials Menu Page

## Adding Menu Page to Save Keys:
-This PHP function adds a menu page titled "Social Login" under the WordPress admin menu. It provides options to manage social login settings and is accessible to users with the "manage_options" capability.

```php
/** Add menu page : Google signup credentials */ 
function add_social_login_menu() {
    add_menu_page(
        'Social Login Settings',
        'Social Login',
        'manage_options',
        'social-login-settings',
        'social_login_page_content',
        'dashicons-admin-users',
        30
    );
}
add_action('admin_menu', 'add_social_login_menu');

```
- This PHP function is used to save social login settings. It retrieves the client ID, secret ID, and redirect URI from the AJAX request, sanitizes them, and saves them to the WordPress options table using the update_option() function. It responds with a success message upon successful saving of settings.
- jQuery AJAX function is used to handle the form submission for saving social login settings.

```php

/** Callback function for Google signup credentials */ 
function social_login_page_content() {
	$client_id = get_option('social_login_client_id');
	$secret_id = get_option('social_login_secret_id');
	$redirect_uri = get_option('social_login_redirect_uri');
	?>
	
	<form id="social-login-form">
	<label for="client-id">Client ID:</label>
	<input type="text" id="client-id" name="client_id" value="<?php echo esc_attr($client_id); ?>"  style="width: 100%;"><br>
	
	<label for="secret-id">Secret ID:</label>
	<input type="text" id="secret-id" name="secret_id" value="<?php echo esc_attr($secret_id); ?>"  style="width: 100%;"><br>
	
	<label for="redirect-uri">Redirect URI:</label>
	<input type="text" id="redirect-uri" name="redirect_uri" value="<?php echo esc_attr($redirect_uri); ?>"  style="width: 100%;"><br>
	
	<button type="submit" id="save-settings" class="button-primary">Save Settings</button>
	</form>
	<div id="status-message"></div>   
    <?php
}
add_action('wp_ajax_save_social_login_settings', 'save_social_login_settings');
function save_social_login_settings() {
    // Sanitize and save settings
    $client_id = $_POST['client_id'];
    $secret_id = $_POST['secret_id'];
    $redirect_uri = $_POST['redirect_uri'];
    
    // Save settings to database or any other action
    update_option('social_login_client_id', $client_id);
    update_option('social_login_secret_id', $secret_id);
    update_option('social_login_redirect_uri', $redirect_uri);
    
    wp_send_json_success('Settings saved successfully');
}
jQuery('#social-login-form').on('submit', function(e) {
    e.preventDefault();
    var client_id = jQuery('#client-id').val();
    var secret_id = jQuery('#secret-id').val();
    var redirect_uri = jQuery('#redirect-uri').val();
    
    jQuery.ajax({
        url: ajax_object.ajax_url,
        type: 'POST',
        data: {
            action: 'save_social_login_settings',
            client_id: client_id,
            secret_id: secret_id,
            redirect_uri: redirect_uri,
        },
        success: function(response) {
            jQuery('#status-message').html('<div class="updated"><p>Settings saved successfully!</p></div>');
            setTimeout(function () {
                jQuery('#status-message').empty();
            }, 2000);               
        },
        error: function(xhr, status, error) {
            jQuery('#status-message').html('<div class="error"><p>' + xhr.responseText + '</p></div>');
        }
    });
});

```
**Copy Client ID, Secret ID, Redirect URI and Save in adminarea**

### Display Google Sign-In Button in Frontend

- This PHP function, google_signin_shortcode(), is designed to create a shortcode for embedding a Google Sign-In button.
```php
function google_signin_shortcode() {
	ob_start();	
	require_once get_template_directory() . '/vendor/autoload.php';
	$client_id = get_option('social_login_client_id');
	$secret_id = get_option('social_login_secret_id');
	$redirect_uri = get_option('social_login_redirect_uri');
	// Initialize Google Client
	$client = new Google_Client();
	$client->setClientId($client_id);
	$client->setClientSecret($secret_id);
	$client->setRedirectUri($redirect_uri);
	$client->addScope("email");
	$client->addScope("profile");
	// Generate login URL
	$loginUrl = $client->createAuthUrl();
	echo '<div class="form_google_link">';
	echo "<a href='$loginUrl'>Continue with Google</a>";
	echo "<p>Or</p>";
	echo "</div>";  
	return ob_get_clean();
}
add_shortcode('google_login', 'google_signin_shortcode');
```
### Google Sign-In And Login Implementation in Frontend

**Ajax Handler for Google Authentication and How to Make user Google register, or Re-Login if already exist and send credential's Mail notification**

```php
/**
 * Ajax handle for google authentication
 */
function google_login_ajax_handler() {
    // Load Google API PHP Client library
    require_once get_template_directory() . '/vendor/autoload.php';

    // Retrieve Google API credentials from WordPress options
    $client_id = get_option('social_login_client_id');
    $secret_id = get_option('social_login_secret_id');
    $redirect_uri = get_option('social_login_redirect_uri');

    // Initialize Google Client
    $client = new Google_Client();
    $client->setClientId($client_id);
    $client->setClientSecret($secret_id);
    $client->setRedirectUri($redirect_uri);
    $client->addScope("email");
    $client->addScope("profile");

    // Handle Google Sign-In callback
    if (isset($_POST['code'])) {
        $token = $client->fetchAccessTokenWithAuthCode($_POST['code']);
        if (!isset($token['error'])) {
            $client->setAccessToken($token['access_token']);
            $google_oauth = new Google_Service_Oauth2($client);
            $google_account_info = $google_oauth->userinfo->get();

            // Check if the user exists in WordPress
            $user_email = $google_account_info->getEmail();
            $first_name = $google_account_info->getGivenName();
            $last_name = $google_account_info->getFamilyName();
            $user = get_user_by('email', $user_email);
            $profile_url = home_url('/profile');
                    
            if (!empty($user->ID)) {
                try {
                    $error = '';
                    if (wp_set_current_user($user->ID)) {
                        // Set authentication cookie
                        if (wp_set_auth_cookie($user->ID)) {
                            // Trigger wp_login action
                            if (do_action('wp_login', $user->user_login)) {
                                
                                // Check if the profile URL is valid
                                if ($profile_url) {
                                    wp_send_json_success(array('redirect' => $profile_url));
                                } else {
                                    $error .= 'Invalid profile URL.';                                   
                                }
                            } else {
                                // Error triggering wp_login action
                                $error = 'Error triggering wp_login action.';                             
                            }
                        } else {
                            $error .= 'Error setting authentication cookie.';                           
                        }
                        wp_send_json_error(array('error' => 'Error triggering wp_login action.','redirect' => $profile_url,'user'=>'exist'));
                    } else {
                        $error .= 'Error setting current user.';
                        // Error setting current user
                        wp_send_json_error(array('error' => 'Error setting current user.'));
                    }
                   
                } catch (Exception $e) {
                    wp_send_json_error(array('error' => $e->getMessage()));
                }
                
            } else {
               // Create user data array
               $password = wp_generate_password();               
               // Create user
               $user_id = wp_create_user($user_email, $password, $user_email);
               if (is_wp_error($user_id)) {
                   // Error creating user
                   $error_message = $user_id->get_error_message();
                   wp_send_json_error(array('error' => $error_message));
               } else {
                    // $user_id = $user->ID;
                    update_user_meta($user_id, 'first_name', $first_name);
                    update_user_meta($user_id, 'last_name', $last_name);

                    // Send notification email to user
                    $subject = 'Welcome to our website';
                    $message = 'Dear ' . $first_name . ',<br><br>';
                    $message .= 'Thank you for signing up on our website. Your account has been created successfully.<br>';
                    $message .= 'Your username: ' . $user_email . '<br>';
                    $message .= 'Your password: ' . $password . '<br><br>';
                    $message .= 'Please keep this information secure.<br><br>';
                    $message .= 'Best regards,<br>';

                    // Send the email
                    $headers[] = 'Content-Type: text/html; charset=UTF-8';
                    wp_mail($user_email, $subject, $message, $headers);

                    // User created successfully, sign in the user
                   $creds = array(
                       'user_login' => $user_email,
                       'user_password' => $password,
                   );
                   $signon_user = wp_signon($creds);
                   if (is_wp_error($signon_user)) {
                       // Error signing in the user
                       $error_message = $signon_user->get_error_message();
                       wp_send_json_error(array('error' => $error_message));
                   } else {
                       // Sign in successful, send success response
                       $redirect_url = home_url('/profile');
                       wp_send_json_success(array('redirect' => $redirect_url));
                   }
               }
            }
        }
    } else {
        // Code parameter not found in the request
        wp_send_json_error(array('error' => 'No code parameter found.'));
    }
}

// Hook the AJAX handler function
add_action('wp_ajax_google_login_ajax', 'google_login_ajax_handler');
add_action('wp_ajax_nopriv_google_login_ajax', 'google_login_ajax_handler');
```
- This function handles the AJAX request for Google authentication.
- It loads the Google API PHP Client library.
- Retrieves Google API credentials from WordPress options.
- Initializes the Google Client with the retrieved credentials and necessary scopes.
- Handles the Google Sign-In callback when a code parameter is present in the AJAX request.

### User Existence Check and Sign-In:
- Checks if the user exists in the WordPress database based on their email.
- If the user exists, attempts to sign them in by setting the current user and authentication cookie.
- If successful, triggers the `wp_login` action and redirects the user to the profile URL.
- If any errors occur during the sign-in process, returns appropriate error messages.

### New User Creation and Sign-In:
- If the user does not exist in the database,
  - it creates a new user using the `wp_create_user()` function with their Google email as both username and email, and a generated password.
  - Updates the newly created user's metadata with their first and last names obtained from Google.
  - Sends a welcome email to the user with their login credentials.
  - Attempts to sign in the newly created user using `wp_signon()`.
  - If successful, redirects the user to the profile URL.
  - If any errors occur during the user creation or sign-in process, returns appropriate error messages.

- Ajax Function 
```php
jQuery(document).ready(function($) {
    // Check if the URL contains the 'code' parameter
    if (window.location.search.includes('code=')) {
        // Extract the 'code' parameter value from the URL
        var urlParams = new URLSearchParams(window.location.search);
        var code = urlParams.get('code');
        jQuery('.small-loading-wrap').show();
        // If the 'code' parameter is present, trigger the AJAX request
        jQuery.ajax({
            type: 'POST',
            url: ajax_object.ajax_url,
            data: {
                action: 'google_login_ajax',
                code: code // Pass the 'code' parameter in the AJAX request data
            },
            success: function(response) {   
                console.log(response);
                // Redirect to the URL received in the AJAX response
                window.location.href = response.data.redirect;
            },
            error: function(xhr, status, error) {
                if (xhr.responseJSON && xhr.responseJSON.user) {
                    // If user exists, redirect
                    window.location.href = xhr.responseJSON.redirect;
                } else {
                    // If there's an error and user doesn't exist, display the error message
                    jQuery('#profile_response').text('Error: ' + error);
                }
                jQuery('.small-loading-wrap').hide();
            }
        });
    }
});   
```
