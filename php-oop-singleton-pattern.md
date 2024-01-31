# Understanding Object Instantiation Patterns in PHP

## Singleton Pattern Example

Consider the following PHP code implementing the Singleton Pattern:

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
