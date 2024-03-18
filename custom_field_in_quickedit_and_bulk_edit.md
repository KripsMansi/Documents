## In this tutorial I will show you how to add any custom fields to WordPress quick edit, Bulk Edit, Custom meta box.
## Add Meta Box
```php
function custom_product_feed_meta_box() {
    add_meta_box(
        'custom-product-feed-meta-box',
        __('Product Feed Settings', 'textdomain'),
        'render_custom_product_feed_meta_box',
        'product', // Change 'product' to your custom post type if needed
        'side',
        'default'
    );
}
add_action('add_meta_boxes', 'custom_product_feed_meta_box');
```
**Render Meta Box Content**
```php
function render_custom_product_feed_meta_box($post) {
    // Add a nonce field so we can check for it later.
    wp_nonce_field('custom_product_feed_meta_box', 'custom_product_feed_meta_box_nonce');

    // Get the current value of the meta field
    $value = get_post_meta($post->ID, '_product_feed_enabled', true);
    // If no value is found, set default to 'on'
    if ($value === '') {
        $value = 'on';
    }    ?>
    <label for="product_feed_enabled">
        <input type="checkbox" name="product_feed_enabled" id="product_feed_enabled" <?php checked($value, 'on'); ?> />
        <?php _e('Enable product in feed', 'textdomain'); ?>
    </label>   <?php
}
```
**Save Meta Box Data**

```php
function save_custom_product_feed_meta_box($post_id) {
    // Check if our nonce is set.
    if (!isset($_POST['custom_product_feed_meta_box_nonce'])) {
        return;
    }

    // Verify that the nonce is valid.
    if (!wp_verify_nonce($_POST['custom_product_feed_meta_box_nonce'], 'custom_product_feed_meta_box')) {
        return;
    }

    // If this is an autosave, our form has not been submitted, so we don't want to do anything.
    if (defined('DOING_AUTOSAVE') && DOING_AUTOSAVE) {
        return;
    }

    // Check the user's permissions.
    if (!current_user_can('edit_post', $post_id)) {
        return;
    }

    // Update the meta field in the database.
    $value = isset($_POST['product_feed_enabled']) ? 'on' : 'off';
    update_post_meta($post_id, '_product_feed_enabled', $value);
}
add_action('save_post', 'save_custom_product_feed_meta_box');
```
##  Add Column to Admin Screen
```php
function custom_product_feed_column($columns) {
    $columns['product_feed'] = __('Product Feed', 'textdomain');
    return $columns;
}
add_filter('manage_product_posts_columns', 'custom_product_feed_column');
// Populate Column with Meta Data and Make it Editable
function custom_product_feed_column_content($column, $post_id) {
    if ($column == 'product_feed') {
        // Get the current value of the meta field
      echo  $value = get_post_meta($post_id, '_product_feed_enabled', true);
        // If no value is found, default to 'on'
        if ($value === '') {
            $value = 'on';
        }

        // Output editable status
        echo '<input type="checkbox" class="product-feed-toggle" data-post-id="' . $post_id . '" data-current-value="' . $value . '" ' . checked($value, 'on', false) . '>';
        echo ($value == 'on') ? __('Enabled', 'textdomain') : __('Disabled', 'textdomain');
    }
}
add_action('manage_product_posts_custom_column', 'custom_product_feed_column_content', 10, 2);
```
## Custom Fields in Bulk Edit

```php
// Add custom checkbox field to bulk edit form
function add_product_feed_bulk_edit_field() {    ?>
    <label>
        <span class="title"><?php _e('Feed', 'textdomain'); ?></span>
        <span class="input-text-wrap">
        <input type="checkbox" name="product_feed_enabled_bulk" class="product-feed-toggle-bulk" /><?php _e('Enable product in feed', 'textdomain'); ?>
        </span>
    </label>  <?php
}
add_action('woocommerce_product_bulk_edit_end', 'add_product_feed_bulk_edit_field');
// Save the custom fields data when submitted for product bulk edit
add_action('woocommerce_product_bulk_edit_save', 'save_custom_field_product_bulk_edit', 10, 1);

function save_custom_field_product_bulk_edit($product) {
   
    $product_id = method_exists($product, 'get_id') ? $product->get_id() : $product->id;

    if (isset($_REQUEST['product_feed_enabled_bulk'])) {
        // Update the custom field meta for the product
        update_post_meta($product_id, '_product_feed_enabled', $_REQUEST['product_feed_enabled_bulk']);
    }
   
}
```
## Custom Fields in Quick Edit
```php
// Add custom checkbox field to quick edit form
function add_product_feed_quick_edit_field($column_name, $post_type) {
    if ($column_name !== 'product_feed' || $post_type !== 'product') {
        return;
    }

    global $post;

    // Get the current value of the meta field
    $product_feed_enabled = get_post_meta($post->ID, '_product_feed_enabled', true);

    ?>
    <fieldset class="inline-edit-col-right">
        <div class="inline-edit-col">
            <label>
                <span class="title"><?php _e('Enable product in feed', 'textdomain'); ?></span>
                <span class="input-text-wrap">
                    <input type="checkbox" name="product_feed_enabled" class="product-feed-toggle">
                    <input type="hidden" name="current_product_feed_enabled" value="<?php echo esc_attr($product_feed_enabled); ?>">
                </span>
            </label>
        </div>
    </fieldset>
    <?php   
}
add_action('quick_edit_custom_box', 'add_product_feed_quick_edit_field', 10, 2);
```

**Save custom checkbox field value in quick edit**
```php
function save_product_feed_quick_edit_field($product_id) {
    if (isset($_REQUEST['product_feed_enabled'])) {
        $product_feed_enabled = $_REQUEST['product_feed_enabled'] === 'on' ? 'on' : 'no';
        update_post_meta($product_id, '_product_feed_enabled', $product_feed_enabled);
    } else {
        // If the checkbox is unchecked, make sure to update the meta value accordingly
        update_post_meta($product_id, '_product_feed_enabled', 'no');
    }
}
add_action('save_post_product', 'save_product_feed_quick_edit_field');
// Add inline script to handle dynamic updating of checkbox in quick edit form
function product_feed_quick_edit_script() {
    global $current_screen;
    if ($current_screen->id === 'edit-product') {
        ?>
        <script>
            jQuery(document).ready(function($) {
                $(document).on('click', '.row-actions .inline', function() {
                    var postId = $(this).closest('tr').attr('id').replace('post-', '');
                    var quickEditRow = $('#edit-' + postId);

                    $.ajax({
                        url: ajaxurl,
                        type: 'POST',
                        data: {
                            action: 'get_product_feed_enabled',
                            post_id: postId
                        },
                        success: function(response) {
                            quickEditRow.find('.product-feed-toggle').prop('checked', response === 'on');
                        }
                    });
                });
            });
        </script>
        <?php
    }
}
add_action('admin_footer', 'product_feed_quick_edit_script');
```
