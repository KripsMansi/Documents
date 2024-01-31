# Understanding Object Instantiation Patterns in PHP
The singleton pattern ensures that there's a single point of access to that instance, usually via a static method or property.

Here's a brief overview of the singleton design pattern and how it might be applied to a WordPress plugin:

   1.Singleton Design Pattern:
        The singleton pattern is a creational design pattern that restricts the instantiation of a class to a single instance and provides a global point of access to that instance.
        It often involves a private constructor, a static method to retrieve the instance, and a private static property to hold the single instance.

   2.Singleton in WordPress Plugin:
        In the context of a WordPress plugin, implementing a singleton pattern can be beneficial in scenarios where you want to ensure that only one instance of the plugin class is created.
        This can be useful for managing global settings, resources, or ensuring that certain actions or filters are hooked into the system only once.

## Singleton Pattern Example

```php
class MySingletonPlugin {

    // Hold the single instance of the class
    private static $instance = null;

    // Private constructor to prevent multiple instances
    private function __construct() {
        // Initialization code here
    }

    // Method to get the single instance of the class
    public static function get_instance() {
        if (null === self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    // Other methods and functionality of the plugin go here
}

// Usage
$singleton_instance = MySingletonPlugin::get_instance();

```
## Let me break down the code for you:

1. private static $instance = null;:
This line declares a private static property named $instance within the class. It is used to store the single instance of the class.
It's static so that it belongs to the class itself rather than an instance of the class.

2. private function __construct() { /* Initialization code here */ }:
This is a private constructor method. It is called when you create a new instance of the class using new self() (as seen in the get_instance method).
The __construct method is often used for initializing properties or performing setup tasks when an object is created.
Making the constructor private ensures that the class cannot be instantiated from outside the class itself.

3. public static function get_instance() { /* ... */ }:
This is a public static method named get_instance. It's used to get the single instance of the class.
It checks if the $instance property is null. If it is, it creates a new instance of the class (new self()) and assigns it to $instance.
If the $instance is already set, it simply returns the existing instance.
This method is static, meaning you can call it without creating an instance of the class. It's a common convention for singleton patterns.

In this line, MySingletonPlugin::get_instance() is called to obtain the single instance of the MySingletonPlugin class. 
If an instance doesn't exist, it will be created using the private constructor. If an instance already exists, the existing instance is returned. 
This ensures that throughout your application, there is only one instance of the MySingletonPlugin class.

## Consider the following PHP code implementing the Singleton Pattern:

```php
class Logger {
    private static $instance = null;
    private $logs = [];

    private function __construct() {}

    public static function get_instance() {
        if (null === self::$instance) {
            self::$instance = new self();
        }
        return self::$instance;
    }

    public function log($message) {
        $this->logs[] = $message;
    }

    public function getLogs() {
        return $this->logs;
    }
}

// Usage
$logger_instance_1 = Logger::get_instance();
$logger_instance_2 = Logger::get_instance();

$logger_instance_1->log("Message from Instance 1");
$logger_instance_2->log("Message from Instance 2");

$logs_1 = $logger_instance_1->getLogs();
$logs_2 = $logger_instance_2->getLogs();

print_r($logs_1); // Output: Array ( [0] => Message from Instance 1, [1] => Message from Instance 2 )
print_r($logs_2); // Output: Array ( [0] => Message from Instance 1, [1] => Message from Instance 2 )

```

## Explanation:

In this example, both $logger_instance_1 and $logger_instance_2 point to the same single instance of the Logger 
class due to the use of the singleton pattern (Logger::get_instance()).Logging a message with one instance affects 
the logs of the other instance because they share the same instance.

## Regular Class Instance Example:
```php
class Logger {
    private $logs = [];

    public function log($message) {
        $this->logs[] = $message;
    }

    public function getLogs() {
        return $this->logs;
    }
}

// Creating multiple instances of the Logger class
$logger_instance_1 = new Logger();
$logger_instance_2 = new Logger();

$logger_instance_1->log("Message from Instance 1");
$logger_instance_2->log("Message from Instance 2");

$logs_1 = $logger_instance_1->getLogs();
$logs_2 = $logger_instance_2->getLogs();

print_r($logs_1); // Output: Array ( [0] => Message from Instance 1 )
print_r($logs_2); // Output: Array ( [0] => Message from Instance 2 )
```
## Explanation:

In this example, $logger_instance_1 and $logger_instance_2 are independent instances of the Logger class, each with its own set of logs.
Logging a message with one instance does not affect the logs of the other instance because they operate independently.

In summary, the singleton pattern creates a single shared instance that is accessed globally, while using regular class instances allows you to
create multiple independent instances of the class, each with its own state. The choice between them depends on your specific use case and whether 
you need shared or independent instances.
