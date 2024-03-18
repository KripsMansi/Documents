### Schedule Cron Event

- The code includes functions to schedule and run the cron job responsible for executing tasks at specified intervals. 
The schedule_feed_cron_job() function schedules the cron job based on the selected recurrence option and feed status.
The run_feed_cron_job() function defines the tasks to be performed when the cron job is triggered, such as displaying an admin notice.
```php
function my_custom_menu_page() {
    add_menu_page(
        'Feed Settings',      // Page title
        'Feed Settings',      // Menu title
        'manage_options',     // Capability required to access
        'feed-settings',      // Menu slug
        'feed_settings_page'  // Callback function to render the page
    );
}
add_action('admin_menu', 'my_custom_menu_page');
function feed_settings_page(){
    ?>
    <style>
    .feed-settings-box {
        border: 1px solid #ccc;
        padding: 20px;
        margin: 20px 0;
        border-radius: 5px;
        background-color: #f9f9f9;
    }

    .feed-settings-box h2 {
        margin-top: 0;
    }

    .feed-settings-box label {
        display: block;
        margin-bottom: 5px;
        font-weight: bold;
    }

    .feed-settings-box select,
    .feed-settings-box input[type="submit"] {
        width: 100%;
        margin-bottom: 15px;
        padding: 8px;
        border: 1px solid #ccc;
        border-radius: 3px;
        box-sizing: border-box;
    }

    .feed-settings-box input[type="submit"] {
        width: auto;
        cursor: pointer;
    }

    #status-message {
        margin-top: 15px;
    }
</style>
<div class="wrap">
    <div class="feed-settings-box">
        <h2>Cron Settings</h2>
        <form id="feed-settings-form">            
            <label for="recurrence">Recurrence:</label>
            <select id="recurrence" name="recurrence">
                <option value="30_minutes" <?php selected(get_option('recurrence'), '30_minutes'); ?>>Every 30 minutes</option>
                <option value="hourly" <?php selected(get_option('recurrence'), 'hourly'); ?>>Hourly</option>
                <option value="daily" <?php selected(get_option('recurrence'), 'daily'); ?>>Once a day</option>
                <option value="twice_daily" <?php selected(get_option('recurrence'), 'twice_daily'); ?>>Twice a day</option>
                <option value="weekly" <?php selected(get_option('recurrence'), 'weekly'); ?>>Once weekly</option>
                <option value="15_days" <?php selected(get_option('recurrence'), '15_days'); ?>>Every 15 days</option>
                <option value="monthly" <?php selected(get_option('recurrence'), 'monthly'); ?>>Once a month</option>
            </select><br>
            <label for="status">Status:</label>
            <select id="status" name="status">
                <option value="on" <?php selected(get_option('feed_status'), 'on'); ?>>On</option>
                <option value="off" <?php selected(get_option('feed_status'), 'off'); ?>>Off</option>
            </select><br>

            <input type="submit" value="Save Settings" class="button button-primary">
        </form>

        <div id="status-message"></div>
    </div>
</div>
<?php
}
//js script
 jQuery('#feed-settings-form').on('submit', function(e) {
        e.preventDefault();
        // alert("test");
        var recurrence = jQuery('#recurrence').val();
        console.log(recurrence);
        // var duration = jQuery('#duration').val();
        var status = jQuery('#status').val();

        jQuery.ajax({
            url: ajax_object.ajax_url,
            type: 'POST',
            data: {
                action: 'save_feed_settings',
                recurrence: recurrence,
                status: status,
            },
            success: function(response) {
                jQuery('#status-message').html('<div class="updated"><p>Settings saved successfully!</p></div>');
            },
            error: function(xhr, status, error) {
                console.error(xhr.responseText);
            }
        });
    });
add_action('wp_ajax_save_feed_settings', 'save_feed_settings');
function save_feed_settings() {
    if (isset($_POST['recurrence'])) {
        update_option('recurrence', sanitize_text_field($_POST['recurrence']));
    }
    if (isset($_POST['status'])) {
        update_option('feed_status', sanitize_text_field($_POST['status']));

        // Pause the cron job by clearing the scheduled hook
        wp_clear_scheduled_hook('run_feed_cron_job');

        // If the status is "on" or "yes", reschedule the cron job
        if (sanitize_text_field($_POST['status']) === 'on' || sanitize_text_field($_POST['status']) === 'yes') {
            schedule_feed_cron_job(); // Schedule cron job after saving settings
        }
    }

    wp_die(); // Always include wp_die() at the end of AJAX callbacks
}

// Schedule the cron job
function schedule_feed_cron_job() {
    $recurrence = get_option('recurrence');
    $feed_status = get_option('feed_status');

    if ($feed_status === 'off') {
        wp_clear_scheduled_hook('run_feed_cron_job');
        return;
    }

    // Define interval based on recurrence option
    if ( ! wp_next_scheduled( 'run_feed_cron_job' ) ) {

        wp_schedule_event( time() + 1, 'run_feed_cron', 'run_feed_cron_job');
    }
}
add_action('init', 'schedule_feed_cron_job');

// Run the script when the scheduled event is triggered
function run_feed_cron_job() {
    // Perform your cron job tasks here
    // For demonstration purposes, we'll just display an admin notice
    add_action('admin_notices', 'feed_cron_success_notice');
}
add_action('run_feed_cron_job', 'run_feed_cron_job');

// Admin notice for cron job success
function feed_cron_success_notice() {
    ?>
    <div class="notice notice-success is-dismissible">
        <p>Cron job ran successfully!</p>
    </div>
    <?php
}
// Define custom cron interval
function add_feed_cron_interval($schedules) {
    $recurrence = get_option('recurrence');
    $interval_30_minutes = 30 * 60; // 30 minutes
    $interval_hourly = 60 * 60; // 1 hour
    $interval_daily = 24 * 60 * 60; // 1 day
    $interval_twice_daily = 12 * 60 * 60; // 12 hours
    $interval_weekly = 7 * 24 * 60 * 60; // 1 week
    $interval_15_days = 15 * 24 * 60 * 60; // 15 days
    $interval_monthly = 30 * 24 * 60 * 60; // 1 month (approximation)
    // Calculate custom intervals based on stored recurrence option
    switch ($recurrence) {
        case '30_minutes':
            $interval = $interval_30_minutes;
            break;
        case 'hourly':
            $interval = $interval_hourly;
            break;
        case 'daily':
            $interval = $interval_daily;
            break;
        case 'twice_daily':
            $interval = $interval_twice_daily;
            break;
        case 'weekly':
            $interval =  $interval_weekly;
            break;
        case '15_days':
            $interval =  $interval_15_days;
            break;
        case 'monthly':
            // Approximating 30 days for monthly interval
            $interval =  $interval_monthly;
            break;
        default:
            // Default to daily interval if recurrence is not recognized
            $interval = $interval_daily;
            break;
    }

    // Add custom interval to the list of available schedules
    $schedules['run_feed_cron'] = array(
        'interval' => $interval,
        'display' => __('Custom Feed Interval')
    );

    return $schedules;
}
add_filter('cron_schedules', 'add_feed_cron_interval');
```
