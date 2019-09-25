# logger_to_kibana

[![Build Status](https://dev.azure.com/ismaelmartinez0550/logger_to_kibana/_apis/build/status/IsmaelMartinez.logger_to_kibana?branchName=master)](https://dev.azure.com/ismaelmartinez0550/logger_to_kibana/_build/latest?definitionId=2&branchName=master)
[![DepShield Badge](https://depshield.sonatype.org/badges/IsmaelMartinez/logger_to_kibana/depshield.svg)](https://depshield.github.io)

---

This project is inteded to generate view from the log messages encountered.

To get the programs help just type:

```bash
python main.py
```

# from here it is out of date. see #23
This returns:

```bash
  Usage: main.py [OPTIONS] COMMAND [ARGS]...

  Process the file provided according to conditions

Options:
  --help  Show this message and exit.

Commands:
  pommands:
  process                    Process the folder
  process_and_generate       Process the folder and generate visualisation
  process_generate_and_send  Process the folder, generate visualisation and
                             send
```

The current available commands are:

## process

Process a folder and prints out the processed functions/logs in the following format:

```bash
[{'type': '<log_type>', 'query': 'message: "<log_filter>"', 'label': '<log_type>: <log_filter>'}]
```

To execute the command run:

```bash
python main.py process -f <folder_location>
```

Check the table under [How does it work] section to get more info about log_type and log_filter.

## process_and_generate

Process a folder (as shown in the process section) and generates a table visualisation for kibana.

To execute the command run:

```bash
python main.py process_and_generate -f <folder_location>
```

## process_generate_and_send

Process a folder, generates a table visualisation for kibana and send it to kibana (currently in localhost:5601)

To execute the command run:

```bash
python main.py process_and_generate -f <folder_location>
```

## How does it work

This program uses different regex `detectors` to filter logs and files to process.

Those can be changed in the [settings.ini](settings.ini) file.

The current available detectors are:

| Detector | Default Value | Propose |
|---|---|---|
| FilesMatchFilter | app/src/**/*.py | Filter the files to process in the provided folder |
| LogDebugDetector | LOG.debug | Detect the log debug message |
| LogDebugFilter | (?<=LOG.debug\(["\']).*?(?=["\']) | Filter the log debug message |
| LogInfoDetector | LOG.info | Detect the log info message |
| LogInfoFilter | (?<=LOG.info\(["\']).*?(?=["\']) | Filter the log info message |
| LogWarnDetector | LOG.warn | Detect the log warn message |
| LogWarnFilter | (?<=LOG.warn\(["\']).*?(?=["\']) | Filter the log warn message |
| LogErrorDetector | LOG.error | Detect the log error message |
| LogErrorFilter | (?<=LOG.error\(["\']).*?(?=["\']) | Filter the log error message |
| LogCriticalDetector | LOG.critical | Detect the log critical message |
| LogCriticalFilter | (?<=LOG.critical\(["\']).*?(?=["\']) | Filter the log critical message |

Other configuration available in the settings.ini file are:
| Type | Value | Propose |
| -- | -- | -- |
| BaseUrl | [http://localhost:5601](http://localhost:5601) | Kibana base url |
| Index | 90943e30-9a47-11e8-b64d-95841ca0b247 | Kibana index |

## The process

The commands for the application are done in the following logical order.

```bash
    process -> generate -> send
```

As an example, when processing the following file:

```python
def lambda_handler(_event: dict, _context):
    LOG.debug('Initialising')
    LOG.info('Processing')
    LOG.warn('Success')
    LOG.error('Failure')
    LOG.critical('Bananas')
)
```

Will return the next object:

```python
[{'type': 'debug', 'query': 'message: "Initialising"', 'label': 'debug: Initialising'}]
[{'type': 'info', 'query': 'message: "Processing"', 'label': 'info: Processing'}]
[{'type': 'warn', 'query': 'message: "Success"', 'label': 'warn: Success'}]
[{'type': 'error', 'query': 'message: "Failure"', 'label': 'error: Failure'}]
[{'type': 'critical', 'query': 'message: "Bananas"', 'label': 'critical: Bananas'}]
```

Generate, will generate a table visualisation with filters for all the logs that have found.

The choise of having a table visualisation is for the amount of information available. It is, basically, a good place to start as later we can split the visualisations by file, function or whatever we choose to do so.

To finish, it sends the generated visualisation to Kibana with the following name format:

## Limitations

Currently, this project does not separate logs per file or function. For that reason, it was choosen to use the table visualisation as it is easy to generate too many filter for the other types of visualisations.

It only generates the visualisations and not the dashboards.
