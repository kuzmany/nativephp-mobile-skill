# Queues — NativePHP Mobile v3.1+

**Source (pinned):** `https://github.com/NativePHP/nativephp.com/blob/393dffc5e070675654b64275d3b44d4923a6635f/resources/views/docs/mobile/3/concepts/queues.md`

## Background Queue Worker

NativePHP 3.1+ runs a background queue worker alongside your app's main thread. Queued jobs execute off the main thread, so they won't block your UI.

Both iOS and Android are supported.

## Setup

```dotenv
QUEUE_CONNECTION=database
```

That's it. NativePHP handles the rest — the worker starts automatically when the app boots. No artisan commands, no supervisor config.

## Usage

Standard Laravel queue dispatching:

```php
use App\Jobs\SyncData;

SyncData::dispatch($payload);
// or
dispatch(new App\Jobs\ProcessUpload($file));
```

### Example Job

```php
namespace App\Jobs;

use Illuminate\Bus\Queueable;
use Illuminate\Contracts\Queue\ShouldQueue;
use Illuminate\Foundation\Bus\Dispatchable;
use Illuminate\Queue\InteractsWithQueue;
use Illuminate\Queue\SerializesModels;
use Illuminate\Support\Facades\Http;
use NativePHP\Plugins\Dialog\Dialog;

class SyncData implements ShouldQueue
{
    use Dispatchable, InteractsWithQueue, Queueable, SerializesModels;

    public function __construct(public array $payload) {}

    public function handle()
    {
        Http::post('https://api.example.com/sync', $this->payload);

        Dialog::toast('Sync complete!');
    }
}
```

## How It Works

At boot, NativePHP starts a dedicated PHP runtime on a separate thread. That worker polls `queue:work --once` in a loop, picking up and running queued jobs as they arrive.

Because the worker runs on its own thread with its own PHP runtime, queued jobs are fully isolated from the main request cycle — long-running tasks won't affect UI responsiveness.

## Constraints

- Requires **ZTS (Thread-Safe) PHP**, included by default in v3.1+
- Only the **`database`** queue connection is supported (uses the same SQLite database as your app)
- Jobs persist to the database, so they survive app restarts
- Laravel's standard retry and failure handling applies

## Common Use Cases

- API sync after user-initiated actions (form submit → dispatch sync job → return to UI immediately)
- Periodic background data refresh
- Image/file uploads that should not block interaction
- Push-triggered work that needs to complete after the notification handler returns
