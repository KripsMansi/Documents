# Custom Instruction Box for WPBakery Page Builder

This custom instruction box element for WPBakery Page Builder includes a rich text area for content and an option to select an image as an icon. The element also defines a shortcode to render the instruction box on the frontend.

## Installation

To add this custom element to your WPBakery Page Builder, add the following code to your theme's `functions.php` file or create a custom plugin.

## Code

```php
/***
 * Create wp bakery - Custom Instruction Box with wp editor
 */
if (function_exists('vc_map')) {
  vc_map(array(
      "name" => "Custom Instruction box",
      "base" => "custom_instruction_element",
      "category" => "My Elements",
      "params" => array(
          array(
              "type" => "textarea_html",
              "heading" => "Content",
              "param_name" => "content",
              "value" => "",
              "description" => "Enter the content for your instruction box.",
          ),
          array(
              'type' => 'attach_image', // Use 'attach_image' type for selecting an image
              'heading' => __('Icon', 'text-domain'),
              'param_name' => 'icon',
              'description' => __('Select an icon image.', 'text-domain')
          ),
      ),
  ));
}

add_shortcode('custom_instruction_element', 'custom_instruction_element_output');

function custom_instruction_element_output($atts, $content = null) {
    $atts = shortcode_atts(array(
        'icon' => '',
    ), $atts, 'custom_instruction_element');

    $icon_html = '';
    if (!empty($atts['icon'])) {
        $icon_url = wp_get_attachment_image_src($atts['icon'], 'full');
        if ($icon_url) {
            $icon_html = '<img src="' . esc_url($icon_url[0]) . '" alt="" class="custom-icon" />';
        }
    }

    $output = '<div class="instruction-box">';
    if ($icon_html) {
        $output .= '<div class="instruction-icon custom_instruction_element_icon">' . $icon_html . '</div>';
    }
    
    $output .= '<div class="instruction-content custom-element-content">' . wp_kses_post($content) . '</div>';
    $output .= '</div>';

    return $output;
}
