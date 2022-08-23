# Laravael-IMAP Example Application

This is an example application to showcase the integration of `webklex/laravel-imap`.

## Table of Contents
- [Requirements](#requirements--preconditions)
- [Installation](#installation--setup)
- [Usage](#usage)
  - [Configuration](#account-setup--configuration)
  - [Idle Command](#idle-command)
  - [Additional](#additional-methods)
- [Security](#security)
- [License](#license)

## Requirements & Preconditions
In this example I'm using a linux based system. This means the commands used below may vary slightly compared to windows.

You need to have the following installed:
- PHP (php-mbstring & php-mcrypt)
- Composer

Additional information can be found here: https://www.php-imap.com/installation

## Installation & Setup
Start by creating a new Laravel project. I'm going to name my "laravel_imap_example".
```bash
composer create-project laravel/laravel laravel_imap_example
```

Next lets install `webklex/laravel-imap`
```bash
composer require webklex/laravel-imap
```

In order to use our own custom config, we'll need to publish it first:
```bash
php artisan vendor:publish --provider="Webklex\IMAP\Providers\LaravelServiceProvider"
```
This will create a new file `config/imap.php`. This file contains all possible package configurations.

## Usage

### Account setup & configuration
To use a default account, I'm going to add the following to my `.env` file:
```text
IMAP_HOST=somehost.com
IMAP_PORT=993
IMAP_ENCRYPTION=ssl
IMAP_VALIDATE_CERT=true
IMAP_USERNAME=user@example.com
IMAP_PASSWORD=secret
IMAP_DEFAULT_ACCOUNT=default
IMAP_PROTOCOL=imap
```

### Idle command
Let's assume you want to listen for new mails and execute some code if a new mail has arrived. We can realize this by using the `IDLE` command provided by the imap protocol.
```bash
php artisan make:command CustomImapIdleCommand
```

This will create a new command class inside `app/Console/Commands` called `CustomImapIdleCommand.php`. All we need is to modify it and implement the `Webklex\IMAP\Commands\ImapIdleCommand` class.
```php
<?php  
  
namespace App\Console\Commands;  
  
use Webklex\IMAP\Commands\ImapIdleCommand;  
use Webklex\PHPIMAP\Message;  
  
class CustomImapIdleCommand extends ImapIdleCommand {  
  
    /**  
     * The name and signature of the console command.
     *
	 * @var string  
     */    
     protected $signature = 'custom_command';  
  
    /**  
     * Holds the account information
     *     
     * @var string|array $account  
     */  
    protected $account = "default";  
  
    /**  
     * Callback used for the idle command and triggered for every new received message
     * @param Message $message  
     */  
    public function onNewMessage(Message $message) {  
        $this->info("New message received: " . $message->subject);  
    }  
}
```


This command should print `New message received: the message subject` as soon as we received a new mail. So lets test it:
```bash
php artisan custom_command
```

..and sure enough, after sending an email with the subject "test message", it gets printed to the output:
```bash
New message received: test message
```

Additional reference for commands:
- https://www.php-imap.com/frameworks/laravel/commands
- https://laravel.com/docs/9.x/artisan#writing-commands

### Additional methods
There are of course more possibilities than just idling.
I'm going to create a new command, called "ImapTest":
```bash
php artisan make:command ImapTest
```

```php
<?php  
  
namespace App\Console\Commands;  
  
use Illuminate\Console\Command;  
use Webklex\IMAP\Facades\Client;  
  
class ImapTest extends Command  
{  
    /**  
     * The name and signature of the console command.
     *
	 * @var string  
     */
     protected $signature = 'imap:test';  
  
    /**  
     * The console command description.
     *
	 * @var string  
     */
     protected $description = 'Command description';  
  
    /**  
     * Execute the console command.
     *
	 * @return int  
     */
     public function handle() {
        $client = Client::account("default");  
        $client->connect();  
  
        /** @var \Webklex\PHPIMAP\Support\FolderCollection $folders */  
        $folders = $client->getFolders(false);  
  
        /** @var \Webklex\PHPIMAP\Folder $folder */  
        foreach($folders as $folder){  
            $this->info("Accessing folder: ".$folder->path);  
  
            $messages = $folder->messages()->all()->limit(3, 0)->get();  
  
            $this->info("Number of messages: ".$messages->count());  
            
            /** @var \Webklex\PHPIMAP\Message $message */  
            foreach ($messages as $message) {  
                $this->info("\tMessage: ".$message->message_id);  
            }  
        }
            
		return 0;  
    }
}
```

This command will use the default account (as defined inside your `.env` and `config/imap.php`) and do the following actions:
- Fetch all folders
- Print the path of each found folder
    - Fetch the first 3 messages from that folder
    - Print the message ID of those messages

It might look something like this if executed:
```bash
Accessing folder: INBOX
Number of messages: 3
        Message: d3a5e91963cb805cee975687d5acb1c6@swift.generated
        Message: dfaa51a35c98a7a36dacfeb979f6bcb6@swift.generated
        Message: e017b96972b6b9d58a0124fc8f0113cd@swift.generated
Accessing folder: INBOX.new
Number of messages: 3
        Message: 912455bb-ab24-d158-826a-6d5edf153d73@google.com
        Message: 912455bb-ab24-d158-826a-6d5edf153d73@google.com
        Message: 912455bb-ab24-d158-826a-6d5edf153d73@google.com
Accessing folder: INBOX.9AL56dEMTTgUKOAz
Number of messages: 0
Accessing folder: INBOX.U9PsHCvXxAffYvie
Number of messages: 0
Accessing folder: INBOX.Trash
Number of messages: 3
        Message: 986dd399-d437-51d4-76f7-51002b94e3a9@google.com
        Message: de47dd45-9f60-6813-3dff-10359629a094@google.com
        Message: 764cbf7f-87a4-9d9b-875e-bc2f98a7b212@google.com
Accessing folder: INBOX.processing
Number of messages: 1
        Message: 114dd8055b05250e4eac1a6a7abf9d5f@swift.generated
Accessing folder: INBOX.Sent
Number of messages: 3
        Message: 333da6d3-9722-105e-2e5a-448f04b5c9d6@somehost.com
        Message: e34336cb4569918f190401d414a83c36@somehost.com
        Message: 977e265d598d88d6243f9c69f24567e1@somehost.com
Accessing folder: INBOX.OzDWCXKV3t241koc
Number of messages: 0
Accessing folder: INBOX.5F3bIVTtBcJEqIVe
Number of messages: 0
Accessing folder: INBOX.8J3rll6eOBWnTxIU
Number of messages: 0
Accessing folder: INBOX.Junk
Number of messages: 1
        Message: fb54d9575018517f32efe66916313b92@s5.somehost.se
Accessing folder: INBOX.Drafts
Number of messages: 1
        Message: 2e2511832ff2e6025ca4fab9ecf8d6e2@somehost.com
Accessing folder: INBOX.test
Number of messages: 3
        Message: da313e175eac0b21d9bf417266b00095@swift.generated
        Message: da313e175eac0b21d9bf417266b00095@swift.generated
        Message: 7bee603a843706a7505db393ff703b72@swift.generated
```



## Security
If you discover any security related issues, please email github@webklex.com instead of using the issue tracker.

## License
The MIT License (MIT). Please see [License File][link-license] for more information.


[link-license]: https://github.com/Webklex/laravel_imap_example/blob/master/LICENSE
