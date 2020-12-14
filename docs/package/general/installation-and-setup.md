---
title: Installation & setup
---

Mailcoach can be installed as a package inside an existing Laravel application. This version of Mailcoach does not come with any user management, instead you can [use a gate check to determine who can access Mailcoach](https://mailcoach.app/docs/v3/package/using-the-ui/authorizing-users).

In order to install Mailcoach, you'll need to [get a license](/docs/v3/app/general/getting-a-license) first.
 
 First, add the `satis.spatie.be` repository in your `composer.json`.

```php
"repositories": [
    {
        "type": "composer",
        "url": "https://satis.spatie.be"
    }
],
```

Next, you need to create a file called `auth.json` and place it either next to the `composer.json` file in your project, or in the composer home directory. You can determine the composer home directory on *nix machines by using this command.

```bash
composer config --list --global | grep home
```

This is the content you should put in `auth.json`:

```php
{
    "http-basic": {
        "satis.spatie.be": {
            "username": "<YOUR-SPATIE.BE-ACCOUNT-EMAIL-ADDRESS-HERE>",
            "password": "<YOUR-LICENSE-KEY-HERE>"
        }
    }
}
```

With the configuration above in place, you'll be able to install the package into your project using this command:

```bash
composer require "spatie/laravel-mailcoach:^3.0.0"
```

## Publish the config file

Optionally, You can publish the config file with this command.

```bash
php artisan vendor:publish --provider="Spatie\Mailcoach\MailcoachServiceProvider" --tag="mailcoach-config"
```

Below is the default content of the config file:

```php
<?php

return [

    /*
     * The mailer used by Mailcoach for password resets and summary emails.
     * Mailcoach will use the default Laravel mailer if this is not set.
     */
    'mailer' => null,

    /*
     * The default mailer used by Mailcoach for sending campaigns.
     */
    'campaign_mailer' => null,

    /*
     * The default mailer used by Mailcoach for confirmation and welcome mails.
     */
    'transactional_mailer' => null,

    /*
     * The date format used on all screens of the UI
     */
    'date_format' => 'Y-m-d H:i',

    /*
     * Replacers are classes that can make replacements in the html of a campaign.
     *
     * You can use a replacer to create placeholders.
     */
    'replacers' => [
        \Spatie\Mailcoach\Support\Replacers\WebviewReplacer::class,
        \Spatie\Mailcoach\Support\Replacers\SubscriberReplacer::class,
        \Spatie\Mailcoach\Support\Replacers\EmailListReplacer::class,
        \Spatie\Mailcoach\Support\Replacers\UnsubscribeUrlReplacer::class,
        \Spatie\Mailcoach\Support\Replacers\CampaignReplacer::class,
    ],

    /**
     * Here you can configure which template editor Mailcoach uses.
     * By default this is a text editor that highlights HTML.
     */
    'editor' => \Spatie\Mailcoach\Support\Editor\TextEditor::class,

    /*
     * Here you can specify which jobs should run on which queues.
     * Use an empty string to use the default queue.
     */
    'perform_on_queue' => [
        'calculate_statistics_job' => 'mailcoach',
        'send_campaign_job' => 'send-campaign',
        'send_mail_job' => 'send-mail',
        'send_test_mail_job' => 'mailcoach',
        'send_welcome_mail_job' => 'mailcoach',
        'process_feedback_job' => 'mailcoach-feedback',
        'import_subscribers_job' => 'mailcoach',
    ],

    /*
     * Here you can specify on which connection Mailcoach's jobs will be dispatched.
     * Leave empty to use the app default's env('QUEUE_CONNECTION')
     */
    'queue_connection' => '',

    /*
     * By default only 10 mails per second will be sent to avoid overwhelming your
     * e-mail sending service. To use this feature you must have Redis installed.
     */
    'throttling' => [
        'enabled' => true,
        'redis_connection_name' => 'default',
        'redis_key' => 'laravel-mailcoach',
        'allowed_number_of_jobs_in_timespan' => 10,
        'timespan_in_seconds' => 1,
        'release_in_seconds' => 5,
        'retry_until_hours' => 24,
    ],

    /*
     * You can customize some of the behavior of this package by using our own custom action.
     * Your custom action should always extend the one of the default ones.
     */
    'actions' => [
        /*
         * Actions concerning campaigns
         */
        'calculate_statistics' => \Spatie\Mailcoach\Actions\Campaigns\CalculateStatisticsAction::class,
        'prepare_email_html' => \Spatie\Mailcoach\Actions\Campaigns\PrepareEmailHtmlAction::class,
        'prepare_subject' => \Spatie\Mailcoach\Actions\Campaigns\PrepareSubjectAction::class,
        'prepare_webview_html' => \Spatie\Mailcoach\Actions\Campaigns\PrepareWebviewHtmlAction::class,
        'convert_html_to_text' => \Spatie\Mailcoach\Actions\Campaigns\ConvertHtmlToTextAction::class,
        'personalize_html' => \Spatie\Mailcoach\Actions\Campaigns\PersonalizeHtmlAction::class,
        'personalize_subject' => \Spatie\Mailcoach\Actions\Campaigns\PersonalizeSubjectAction::class,
        'retry_sending_failed_sends' => \Spatie\Mailcoach\Actions\Campaigns\RetrySendingFailedSendsAction::class,
        'send_campaign' => \Spatie\Mailcoach\Actions\Campaigns\SendCampaignAction::class,
        'send_mail' => \Spatie\Mailcoach\Actions\Campaigns\SendMailAction::class,
        'send_test_mail' => \Spatie\Mailcoach\Actions\Campaigns\SendTestMailAction::class,

        /*
         * Actions concerning subscribers
         */
        'confirm_subscriber' => \Spatie\Mailcoach\Actions\Subscribers\ConfirmSubscriberAction::class,
        'create_subscriber' => \Spatie\Mailcoach\Actions\Subscribers\CreateSubscriberAction::class,
        'delete_subscriber' => \Spatie\Mailcoach\Actions\Subscribers\DeleteSubscriberAction::class,
        'import_subscribers' => \Spatie\Mailcoach\Actions\Subscribers\ImportSubscribersAction::class,
        'send_confirm_subscriber_mail' => \Spatie\Mailcoach\Actions\Subscribers\SendConfirmSubscriberMailAction::class,
        'send_welcome_mail' => \Spatie\Mailcoach\Actions\Subscribers\SendWelcomeMailAction::class,
        'update_subscriber' => \Spatie\Mailcoach\Actions\Subscribers\UpdateSubscriberAction::class,
    ],

    /*
     * Unauthorized users will get redirected to this route.
     */
    'redirect_unauthorized_users_to_route' => 'login',

    /*
     *  This configuration option defines the authentication guard that will
     *  be used to protect your the Mailcoach UI. This option should match one
     *  of the authentication guards defined in the "auth" config file.
     */
    'guard' => env('MAILCOACH_GUARD', null),

    /*
     *  These middleware will be assigned to every Mailcoach routes, giving you the chance
     *  to add your own middleware to this stack or override any of the existing middleware.
     */
    'middleware' => [
        'web' => [
            'web',
            Spatie\Mailcoach\Http\App\Middleware\Authenticate::class,
            Spatie\Mailcoach\Http\App\Middleware\Authorize::class,
            Spatie\Mailcoach\Http\App\Middleware\SetMailcoachDefaults::class,
        ],
        'api' => [
            'api',
            'auth:api',
        ],
    ],

    /*
     * This disk will be used to store files regarding importing subscribers.
     */
    'import_subscribers_disk' => 'public',


    'models' => [

        /*
         * The model you want to use as a Campaign model. It needs to be or
         * extend the `Spatie\Mailcoach\Models\Campaign` model.
         */
        'campaign' => Spatie\Mailcoach\Models\Campaign::class,

        /*
         * The model you want to use as a EmailList model. It needs to be or
         * extend the `Spatie\Mailcoach\Models\EmailList` model.
         */
        'email_list' => Spatie\Mailcoach\Models\EmailList::class,

        /*
         * The model you want to use as a Subscriber model. It needs to be or
         * extend the `Spatie\Mailcoach\Models\Subscriber` model.
         */
        'subscriber' => Spatie\Mailcoach\Models\Subscriber::class,

        /*
         * The model you want to use as a Template model. It needs to be or
         * extend the `Spatie\Mailcoach\Models\Template` model.
         */
        'template' => Spatie\Mailcoach\Models\Template::class,

    ],

    'views' => [

        /*
         * The service provider registers several Blade components that are
         * used in Mailcoach's views. If you are using the default Mailcoach
         * views, leave this as true so they work as expected. If you have
         * your own views and don't need/want Mailcoach to register these
         * blade components (e.g., because of naming conflicts), you can
         * change this setting to false and they won't be registered.
         *
         * If you change this setting, be sure to run `php artisan view:clear`
         * so Laravel can recompile your views.
         */
        'use_blade_components' => true,
    ],
];
```

## Configure an email sending service

Mailcoach can send out mail via various email sending services and can handle the feedback (opens, clicks, bounces, complaints) those services give.

Head over to the Laravel documentation to learn [how to set up a mailer](https://laravel.com/docs/7.x/mail#configuration).

Mailcoach can use different mailers for sending confirmation & welcome mails and campaign mails. This way you can keep very good reputations with the mailer service you use for campaign mails, as the confirmation mails might bounce.

To use different mailers, fill in the name of configured mailers in the `campaign_mailer` and `transactional_mailers` of the `mailcoach.php` config file.

To configure tracking open, clicks, bounces & complaints, follow the instruction on the dedicated docs page of each supported service.

- [Amazon SES](/docs/v3/package/handling-feedback/amazon-ses)
- [Mailgun](/docs/v3/package/handling-feedback/mailgun)
- [Sendgrid](/docs/v3/package/handling-feedback/sendgrid)
- [Postmark](/docs/v3/package/handling-feedback/postmark)

## Prepare the database

You need to publish and run the migration:

```bash
php artisan vendor:publish --provider="Spatie\Mailcoach\MailcoachServiceProvider" --tag="mailcoach-migrations"
php artisan migrate
```

## Add the route macro

You must register the routes needed to handle subscription confirmations, open, and click tracking. We recommend that you don't put this in your routes file, but in the `boot` method of `RouteServiceProvider`

```php
Route::mailcoach('mailcoach');
```

## Schedule the commands

In the console kernel, you should schedule these commands.

Regarding the mailcoach:cleanup-processed-feedback command, be aware that email providers such as SES are a 'deliver at least once' service. Duplicate feedback delivery could be seen weeks after the event. Mailcoach prevents duplicates from SES by checking for old matching feedback. As such, cleaning up historical feedback webhooks could lead to duplicate feedbacks items being processed multiple times. The end result is inflated open and click metrics.

```php
// in app/Console/Kernel.php
protected function schedule(Schedule $schedule)
{
    // ...
    $schedule->command('mailcoach:calculate-statistics')->everyMinute();
    $schedule->command('mailcoach:send-scheduled-campaigns')->everyMinute();
    $schedule->command('mailcoach:send-campaign-summary-mail')->hourly();
    $schedule->command('mailcoach:send-email-list-summary-mail')->mondays()->at('9:00');
    $schedule->command('mailcoach:delete-old-unconfirmed-subscribers')->daily();
    $schedule->command('mailcoach:cleanup-processed-feedback')->hourly();
}
```

## Publish the assets

You must publish the JavaScript and CSS assets using this command:

```bash
php artisan vendor:publish --tag mailcoach-assets --force
```

To ensure that these assets get republished each time Mailcoach is updated, we highly recommend you add the following command to the `post-update-cmd` of the `scripts` section of your `composer.json`.

```php
    "scripts": {
        "post-update-cmd": [
            "@php artisan vendor:publish --tag mailcoach-assets --force"
        ]
    }
```

## Install and configure redis

It's common for e-mail providers to limit the number of e-mails you can send within a given amount of time. The package uses Redis to throttle e-mails, so make sure it's available on your system. You must specify a valid Redis connection name in the `throttling.redis_connection_name` key.

By default, we set this value to the default Laravel connection name, `default`.

## Install Horizon

This package handles various tasks in a queued way via [Laravel Horizon](https://laravel.com/docs/horizon). If your application doesn't have Horizon installed yet, follow [their installation instructions](https://laravel.com/docs/horizon#installation).

After Horizon is installed, don't forget to set `QUEUE_CONNECTION` in your `.env` file to `redis`.

`config/horizon.php` should have been created in your project. In this config file, you must add a block named `mailcoach-general` and `mailcoach-heavy` to both the `production` and `local` environment.

```php
'environments' => [
    'production' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
            'tries' => 2,
            'timeout' => 60 * 60,
        ],
        'mailcoach-general' => [
            'connection' => 'mailcoach-redis',
            'queue' => ['mailcoach', 'mailcoach-feedback', 'send-mail'],
            'balance' => 'auto',
            'processes' => 10,
            'tries' => 2,
            'timeout' => 60 * 60,
        ],
        'mailcoach-heavy' => [
            'connection' => 'mailcoach-redis',
            'queue' => ['send-campaign'],
            'balance' => 'auto',
            'processes' => 3,
            'tries' => 1,
            'timeout' => 60 * 60,
        ],
    ],

    'local' => [
        'supervisor-1' => [
            'connection' => 'redis',
            'queue' => ['default'],
            'balance' => 'simple',
            'processes' => 10,
            'tries' => 2,
            'timeout' => 60 * 60,
        ],
        'mailcoach-general' => [
            'connection' => 'mailcoach-redis',
            'queue' => ['mailcoach', 'mailcoach-feedback', 'send-mail'],
            'balance' => 'auto',
            'processes' => 10,
            'tries' => 2,
            'timeout' => 60 * 60,
        ],
        'mailcoach-heavy' => [
            'connection' => 'mailcoach-redis',
            'queue' => ['send-campaign'],
            'balance' => 'auto',
            'processes' => 3,
            'tries' => 1,
            'timeout' => 60 * 60,
        ],
    ],
],
```

In the `config/queue.php` file you must add the `mailcoach-redis` connection:

```php
'connections' => [

    // ...

    'mailcoach-redis' => [
        'driver' => 'redis',
        'connection' => 'default',
        'queue' => env('REDIS_QUEUE', 'default'),
        'retry_after' => 11 * 60,
        'block_for' => null,
    ],
```

## Add authorization to Mailcoach UI

By default, you can only use Mailcoach UI on a local environment. To use it in another environment, you need to register a gate check. Head over to [the section on authorizing users](/docs/v3/package/using-the-ui/authorizing-users) to learn more.

Mailcoach will redirect unauthorized users to the route specified in the `redirect_unauthorized_users_to_route` key of the Mailcoach config file. By default, the `login` route is used. If you don't have a `login` route in your application, set this setting to a route that does exist.
 
## Specifying a timezone

By default, all dates in Mailcoach are in UTC. If you want to use another timezone, you can change the `timezone` setting in the `config/app.php` config file.

## Choosing an editor

By default, Mailcoach uses a plain textarea field to edit campaigns and templates. If you'd like to use a feature rich editor, you can [use one of the add-on packages](/docs/v3/package/customizing-the-editor/introduction).

## Visit the UI

After performing all these steps, you should be able to visit the Mailcoach UI at `/mailcoach`.

Before sending out a real campaign, we highly recommend creating a small email list with a couple of test email addresses and send a campaign to it. This way, you can verify that sending mails, and the open & click tracking are all working correctly.
