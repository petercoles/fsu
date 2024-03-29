# Flumine Strategy Utilities

A collection of utilities whose objective is to aid and DRY the development of Flumine strategies. Each utility is designed to be stand-alone, re-usable and easily integrated into a Flumine strategy or the code used to run that strategy, replacing pieces of code that are often used across multiple Flumine instances.

## Process Logging

It's in the very nature of streaming to produce huge numbers of events. Flumine being event-driven, amplifies this. When something goes wrong, we look to our process log files to find the issue and some clue as to its cause. The process logger (not to be confused with logging controls) can be initialised using the cunningly-named ```initialise_process_logger``` function.

It has a number of options. You can:

* set the *level* at which logging messages will be actioned (default CRITICAL)
* *stream* log entries to the console / terminal (default False)
* persist log entries to a *logfile* (default None)
* just in case you want to use multiple loggers, assign a *name* to the logger being initialised (default None)

Example usage:

``` python
from fsu.process_logger import initialise_logger

logger = initialise_logger(
    level=logging.DEBUG,
    stream=True,
    logfile="path/to/logfile",
    name="myLogger"
)

# log something important
logger.info("Yay, it's working")
```

## Logging Control

Flumine's logging control features are very useful for plugging into key stages of a strategy's execution as it processes markets. The logging control offered by this package, allow the capture of data at a market or individual order level. A particular useful use case is when simulating either backtesting over historical data, or capturing the results of paper trading in real-time.

This example is simular to the one in the Flumine repository's examples, but more extensive in the data that it collects. It also creates the logging control files if they exist. Note that by default it will overwrite the files if they already exist. This by design, so add version numbers or include a timestamp in the control filenames if you want to repeat the runs with different inputs or use the append flag explained below.

The control is designed primarily to capture order data, so the path to an orders log file is mandatory. Optional paramters are a path to a markets log file for the profit / loss per market and a boolean flag to indicate whether you wish to append to an existing log file of the same name. This is especially important when runninig 

Example Usage:

``` python
from fsu.logging-controls.FileLoggingControlWithProfits import FileLoggingControlWithProfits

framework.add_logging_control(
    FileLoggingControlWithProfits(
        orders_file="path/to-orders-log-file" # mandatory
        markets_file="path/to/markets-log-file" # optional
        append_to_logs=True # optional
    )
)
```

## Middleware

Flumine's middleware is a great place to pre-process market data before passing it to one or more strategies running on a Flmine instance. The first piece of middleware included with this package (others will likely follow) is intended for use when backtesting and injects the market catalogue into the market object. It's similar to the version included in Flumine's examples, but is a little different in that Flumine's example assumes filenames ending with the .json extension, whereas this implementation allows the file extension to be passed as a parameter. More subtlely, this implementation by-passes Flumine's events system.

The only mandatory parameter is the path to the folder where the catalogues are being stored. Optionally you can pass a second paramter for the file extension (default: "gz"), wich will be appended to the market_id to form the catalogue filename.

Example Usage:

``` python
from fsu.middleware.market-cataogue import MarketCatalogueMiddleware

framework.add_market_middleware(
    catalogue_path=MarketCatalogueMiddleware("path/to/catalogues/folder"), # mandatory
    file_extension="json.gz", # optional, default "gz"
)
```

## Workers

Workers allow Flumine to spawn additional threads, which the underlying operating system will often place on other processors (where available). The first worker added to this package is a copy of the Flumine terminate worker that can be used to close down a framework instance when there are no further markets to be processed.

All Flumine workers have the same parameters, so the key information that is worker dependent is passed in the func_kwargs parameter as shown in the example below.

Example Usage:

``` python
from fsu.workers.terminate import terminate

framework.add_worker(
    BackgroundWorker(
        framework,
        terminate,
        func_kwargs={"today_only": True, "seconds_closed": 1200},
        interval=60,
        start_delay=60,
    )
)
```
