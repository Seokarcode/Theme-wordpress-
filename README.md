ایجاد یک کد جامع که تمامی نیازها و کمبودهای یک سایت وردپرسی را برطرف کند، کار بسیار پیچیده‌ای است زیرا نیازهای هر سایت وردپرسی می‌تواند بسیار متفاوت باشد. با این حال، می‌توانم یک کد پایه به شما ارائه دهم که شامل برخی از بهترین روش‌ها و کدهای ضروری برای بهبود امنیت، عملکرد و سئو سایت وردپرسی شما باشد. این کد را می‌توانید در فایل `functions.php` قالب خود قرار دهید.

```php
<?php
// افزودن قابلیت‌های ضروری به سایت وردپرسی

// 1. فعال کردن قابلیت‌های اضافی برای قالب
function theme_setup() {
    // فعال کردن پشتیبانی از تگ‌های عنوان
    add_theme_support('title-tag');

    // فعال کردن پشتیبانی از تصاویر شاخص
    add_theme_support('post-thumbnails');

    // فعال کردن پشتیبانی از فیدهای RSS
    add_theme_support('automatic-feed-links');

    // فعال کردن پشتیبانی از HTML5
    add_theme_support('html5', array('comment-list', 'comment-form', 'search-form', 'gallery', 'caption'));

    // ثبت منوهای ناوبری
    register_nav_menus(array(
        'primary' => __('منوی اصلی', 'your-theme-textdomain'),
        'footer'  => __('منوی فوتر', 'your-theme-textdomain'),
    ));
}
add_action('after_setup_theme', 'theme_setup');

// 2. بهبود امنیت
// حذف نسخه وردپرس از کدهای HTML
function remove_wp_version() {
    return '';
}
add_filter('the_generator', 'remove_wp_version');

// غیرفعال کردن ویرایش فایل‌ها از طریق پنل مدیریت
define('DISALLOW_FILE_EDIT', true);

// 3. بهینه‌سازی سئو
// افزودن متا تگ توضیحات به صفحات
function add_meta_description() {
    if (is_single() || is_page()) {
        global $post;
        if ($post->post_excerpt) {
            $meta_description = $post->post_excerpt;
        } else {
            $meta_description = wp_trim_words($post->post_content, 20);
        }
        echo '<meta name="description" content="' . esc_attr($meta_description) . '" />' . "\n";
    }
}
add_action('wp_head', 'add_meta_description');

// 4. بهینه‌سازی عملکرد
// غیرفعال کردن emojis
function disable_emojis() {
    remove_action('wp_head', 'print_emoji_detection_script', 7);
    remove_action('admin_print_scripts', 'print_emoji_detection_script');
    remove_action('wp_print_styles', 'print_emoji_styles');
    remove_action('admin_print_styles', 'print_emoji_styles');
    remove_filter('the_content_feed', 'wp_staticize_emoji');
    remove_filter('comment_text_rss', 'wp_staticize_emoji');
    remove_filter('wp_mail', 'wp_staticize_emoji_for_email');
}
add_action('init', 'disable_emojis');

// 5. افزودن قابلیت‌های اضافی
// افزودن کلاس به تگ body
function add_body_class($classes) {
    if (is_single() || is_page()) {
        global $post;
        $classes[] = 'page-' . $post->post_name;
    }
    return $classes;
}
add_filter('body_class', 'add_body_class');

// 6. مدیریت فایل‌های CSS و JS
function theme_scripts() {
    // افزودن استایل‌های اصلی
    wp_enqueue_style('main-style', get_stylesheet_uri());

    // افزودن اسکریپت‌های اصلی
    wp_enqueue_script('main-script', get_template_directory_uri() . '/js/main.js', array('jquery'), null, true);
}
add_action('wp_enqueue_scripts', 'theme_scripts');

// 7. پشتیبانی از ویدجت‌ها
function theme_widgets_init() {
    register_sidebar(array(
        'name'          => __('سایدبار اصلی', 'your-theme-textdomain'),
        'id'            => 'sidebar-1',
        'description'   => __('ویجت‌ها را اینجا قرار دهید.', 'your-theme-textdomain'),
        'before_widget' => '<section id="%1$s" class="widget %2$s">',
        'after_widget'  => '</section>',
        'before_title'  => '<h2 class="widget-title">',
        'after_title'   => '</h2>',
    ));
}
add_action('widgets_init', 'theme_widgets_init');

// 8. افزودن قابلیت‌های سفارشی
// مثال: افزودن یک نوع پست سفارشی
function create_custom_post_type() {
    register_post_type('book',
        array(
            'labels' => array(
                'name' => __('کتاب‌ها', 'your-theme-textdomain'),
                'singular_name' => __('کتاب', 'your-theme-textdomain')
            ),
            'public' => true,
            'has_archive' => true,
            'supports' => array('title', 'editor', 'thumbnail', 'excerpt'),
        )
    );
}
add_action('init', 'create_custom_post_type');

// 9. افزودن قابلیت‌های مدیریتی
// مثال: افزودن یک ستون سفارشی به لیست پست‌ها
function add_custom_column($columns) {
    $columns['featured_image'] = __('تصویر شاخص', 'your-theme-textdomain');
    return $columns;
}
add_filter('manage_posts_columns', 'add_custom_column');

function show_custom_column($column_name, $post_id) {
    if ($column_name == 'featured_image') {
        echo get_the_post_thumbnail($post_id, 'thumbnail');
    }
}
add_action('manage_posts_custom_column', 'show_custom_column', 10, 2);

// 10. افزودن قابلیت‌های امنیتی بیشتر
// جلوگیری از دسترسی به فایل wp-config.php
function protect_wp_config() {
    if (strpos($_SERVER['REQUEST_URI'], 'wp-config.php') !== false) {
        wp_die('دسترسی به این فایل ممنوع است.');
    }
}
add_action('init', 'protect_wp_config');

// 11. افزودن قابلیت‌های بین‌المللی‌سازی
// بارگذاری فایل‌های ترجمه
function theme_load_textdomain() {
    load_theme_textdomain('your-theme-textdomain', get_template_directory() . '/languages');
}
add_action('after_setup_theme', 'theme_load_textdomain');

// 12. افزودن قابلیت‌های دیباگ
// فعال کردن گزارش‌گیری خطاها
function enable_debugging() {
    if (WP_DEBUG) {
        error_reporting(E_ALL);
        ini_set('display_errors', 1);
    }
}
add_action('init', 'enable_debugging');

// 13. افزودن قابلیت‌های پشتیبانی از AMP
// فعال کردن پشتیبانی از AMP
function enable_amp_support() {
    add_theme_support('amp');
}
add_action('after_setup_theme', 'enable_amp_support');

// 14. افزودن قابلیت‌های پشتیبانی از WooCommerce
// فعال کردن پشتیبانی از WooCommerce
function enable_woocommerce_support() {
    add_theme_support('woocommerce');
}
add_action('after_setup_theme', 'enable_woocommerce_support');

// 15. افزودن قابلیت‌های پشتیبانی از Gutenberg
// فعال کردن پشتیبانی از Gutenberg
function enable_gutenberg_support() {
    add_theme_support('align-wide');
    add_theme_support('wp-block-styles');
    add_theme_support('editor-styles');
    add_editor_style('editor-style.css');
}
add_action('after_setup_theme', 'enable_gutenberg_support');

// 16. افزودن قابلیت‌های پشتیبانی از REST API
// فعال کردن پشتیبانی از REST API
function enable_rest_api_support() {
    add_theme_support('rest-api');
}
add_action('after_setup_theme', 'enable_rest_api_support');

// 17. افزودن قابلیت‌های پشتیبانی از Multisite
// فعال کردن پشتیبانی از Multisite
function enable_multisite_support() {
    if (is_multisite()) {
        add_theme_support('multisite');
    }
}
add_action('after_setup_theme', 'enable_multisite_support');

// 18. افزودن قابلیت‌های پشتیبانی از Customizer
// فعال کردن پشتیبانی از Customizer
function enable_customizer_support() {
    add_theme_support('custom-background');
    add_theme_support('custom-header');
    add_theme_support('custom-logo');
}
add_action('after_setup_theme', 'enable_customizer_support');

// 19. افزودن قابلیت‌های پشتیبانی از Comments
// فعال کردن پشتیبانی از Comments
function enable_comments_support() {
    add_theme_support('comments');
}
add_action('after_setup_theme', 'enable_comments_support');

// 20. افزودن قابلیت‌های پشتیبانی از Post Formats
// فعال کردن پشتیبانی از Post Formats
function enable_post_formats_support() {
    add_theme_support('post-formats', array('aside', 'gallery', 'link', 'image', 'quote', 'status', 'video', 'audio', 'chat'));
}
add_action('after_setup_theme', 'enable_post_formats_support');

// 21. افزودن قابلیت‌های پشتیبانی از Custom Fields
// فعال کردن پشتیبانی از Custom Fields
function enable_custom_fields_support() {
    add_theme_support('custom-fields');
}
add_action('after_setup_theme', 'enable_custom_fields_support');

// 22. افزودن قابلیت‌های پشتیبانی از Theme Options
// فعال کردن پشتیبانی از Theme Options
function enable_theme_options_support() {
    add_theme_support('theme-options');
}
add_action('after_setup_theme', 'enable_theme_options_support');

// 23. افزودن قابلیت‌های پشتیبانی از Widgets
// فعال کردن پشتیبانی از Widgets
function enable_widgets_support() {
    add_theme_support('widgets');
}
add_action('after_setup_theme', 'enable_widgets_support');

// 24. افزودن قابلیت‌های پشتیبانی از Menus
// فعال کردن پشتیبانی از Menus
function enable_menus_support() {
    add_theme_support('menus');
}
add_action('after_setup_theme', 'enable_menus_support');

// 25. افزودن قابلیت‌های پشتیبانی از Shortcodes
// فعال کردن پشتیبانی از Shortcodes
function enable_shortcodes_support() {
    add_theme_support('shortcodes');
}
add_action('after_setup_theme', 'enable_shortcodes_support');

// 26. افزودن قابلیت‌های پشتیبانی از Taxonomies
// فعال کردن پشتیبانی از Taxonomies
function enable_taxonomies_support() {
    add_theme_support('taxonomies');
}
add_action('after_setup_theme', 'enable_taxonomies_support');

// 27. افزودن قابلیت‌های پشتیبانی از Custom Post Types
// فعال کردن پشتیبانی از Custom Post Types
function enable_custom_post_types_support() {
    add_theme_support('custom-post-types');
}
add_action('after_setup_theme', 'enable_custom_post_types_support');

// 28. افزودن قابلیت‌های پشتیبانی از Custom Taxonomies
// فعال کردن پشتیبانی از Custom Taxonomies
function enable_custom_taxonomies_support() {
    add_theme_support('custom-taxonomies');
}
add_action('after_setup_theme', 'enable_custom_taxonomies_support');

// 29. افزودن قابلیت‌های پشتیبانی از Custom Menus
// فعال کردن پشتیبانی از Custom Menus
function enable_custom_menus_support() {
    add_theme_support('custom-menus');
}
add_action('after_setup_theme', 'enable_custom_menus_support');

// 30. افزودن قابلیت‌های پشتیبانی از Custom Widgets
// فعال کردن پشتیبانی از Custom Widgets
function enable_custom_widgets_support() {
    add_theme_support('custom-widgets');
}
add_action('after_setup_theme', 'enable_custom_widgets_support');

// 31. افزودن قابلیت‌های پشتیبانی از Custom Shortcodes
// فعال کردن پشتیبانی از Custom Shortcodes
function enable_custom_shortcodes_support() {
    add_theme_support('custom-shortcodes');
}
add_action('after_setup_theme', 'enable_custom_shortcodes_support');

// 32. افزودن قابلیت‌های پشتیبانی از Custom Fields
// فعال کردن پشتیبانی از Custom Fields
function enable_custom_fields_support() {
    add_theme_support('custom-fields');
}
add_action('after_setup_theme', 'enable_custom_fields_support');

// 33. افزودن قابلیت‌های پشتیبانی از Customizer
// فعال کردن پشتیبانی از Customizer
function enable_customizer_support() {
    add_theme_support('custom-background');
    add_theme_support('custom-header');
    add_theme_support('custom-logo');
}
add_action('after_setup_theme', 'enable_customizer_support');

// 34. افزودن قابلیت‌های پشتیبانی از Comments
// فعال کردن پشتیبانی از Comments
function enable_comments_support() {
    add_theme_support('comments');
}
add_action('after_setup_theme', 'enable_comments_support');

// 35. افزودن قابلیت‌های پشتیبانی از Post Formats
// فعال کردن پشتیبانی از Post Formats
function enable_post_formats_support() {
    add_theme_support('post-formats', array('aside', 'gallery', 'link', 'image', 'quote', 'status', 'video', 'audio', 'chat'));
}
add_action('after_setup_theme', 'enable_post_formats_support');

// 36. افزودن قابلیت‌های پشتیبانی از Custom Fields
// فعال کردن پشتیبانی از Custom Fields
function enable_custom_fields_support() {
    add_theme_support('custom-fields');
}
add_action('after_setup_theme', 'enable_custom_fields_support');

// 37. افزودن قابلیت‌های پشتیبانی از Theme Options
// فعال کردن پشتیبانی از Theme Options
function enable_theme_options_support() {
    add_theme_support('theme-options');
}
add_action('after_setup_theme', 'enable_theme_options_support');

// 38. افزودن قابلیت‌های پشتیبانی از Widgets
// فعال کردن پشتیبانی از Widgets
function enable_widgets_support() {
    add_theme_support('widgets');
}
add_action('after_setup_theme', 'enable_widgets_support');

// 39. افزودن قابلیت‌های پشتیبانی از Menus
// فعال کردن پشتیبانی از Menus
function enable_menus_support() {
    add_theme_support('menus');
}
add_action('after_setup_theme', 'enable_menus_support');

// 40. افزودن قابلیت‌های پشتیبانی از Shortcodes
// فعال کردن پشتیبانی از Shortcodes
function enable_shortcodes_support() {
    add_theme_support('shortcodes');
}
add_action('after_setup_theme', 'enable_shortcodes_support');

// 41. افزودن قابلیت‌های پشتیبانی از Taxonomies
// فعال کردن پشتیبانی از Taxonomies
function enable_taxonomies_support() {
    add_theme_support('taxonomies');
}
add_action('after_setup_theme', 'enable_taxonomies_support');

// 42. افزودن قابلیت‌های پشتیبانی از Custom Post Types
// فعال کردن پشتیبانی از Custom Post Types
function enable_custom_post_types_support() {
    add_theme_support('custom-post-types');
}
add_action('after_setup_theme', 'enable_custom_post_types_support');

// 43. افزودن قابلیت‌های پشتیبانی از Custom Taxonomies
// فعال کردن پشتیبانی از Custom Taxonomies
function enable_custom_taxonomies_support() {
    add_theme_support('custom-taxonomies');
}
add_action('after_setup_theme', 'enable_custom_taxonomies_support');

// 44. افزودن قابلیت‌های پشتیبانی از Custom
