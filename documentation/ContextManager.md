# Context Manager

## Description
The Context Manager is responsible for evaluating the current context against the configuration instructions given by the matchmaker (-framework) and pass the resulting configuration on to the lifecycle manager, PCP, etc., via events on the flowmanager.

The Context Manager is active on the following times in the system.
* **When the matchmaking process has finished**, the context manager will evaluate the data that has been output from the matchmaker against the current context and pass the result on to other components via events on the flowmanager.
* **When the context changes** the Context Manager will re-evaluate the last matchmaker output against the new context. If the change in context means that the system needs to be re-configured, the new configuration will be sent to the relevant component via events on the flowmanager.
 
##API

### Reporting Changes in Environment (PUT /environmentChanged)
* **description**: For environment reporters to notify the system that the environment has changed
* **Supported modes**: works only with a locally installed GPII framework (i.e. non-cloud-based flowmanager)
* **route:** `/environmentChanged`
* **method:** `PUT`
* **data format:** The data of the PUT request should be a JSON object, containing at least a timestamp key and one or more environment contexts. The timestamp should be in the ISO_8601 format (e.g "2014-10-10T09:59:01.0000123+02:00"). Examples of payloads to the PUT request are the following:

```
{
    "timestamp": "2014-10-10T09:59:01.0000123+02:00",
    "http://registry.gpii.net/common/environment/illuminance": 762,
    "http://registry.gpii.net/common/environment/auditory.noise": -40
}
```

```
 {
    "timestamp": "2014-23-12T09:59:01.01123923+01:00",
    "http://registry.gpii.net/common/environment/timeOfDay": "23:15"
}
```

## Testing:

To test the context manager's handling of environment changes, use CURL to send a POST request with the desired payload, e.g.:

```
curl  -H "Content-Type: application/json" -X PUT -d '{"timestamp": "2014-23-12T09:59:01.01123923+01:00","http://registry.gpii.net/common/environment/illuminance": 122}' http://localhost:8081/environmentChanged
```

## Supported Contexts and Condition Transformations

### Supported Condition Transformations:

Conditions in needs and preferences sets can come from two different sources. They can either be generated by one of the match makers, or be manually created by the user via a preferences editor.

The conditions supported by the system are: 
* `http://registry.gpii.net/conditions/inRange`
* `http://registry.gpii.net/conditions/timeInRange`

These conditions along with their expected inputs are described below:

#### http://registry.gpii.net/conditions/inRange

This condition is declared using the `http://registry.gpii.net/conditions/inRange` type. It can be used for any condition which deals with a range in which a context value has to fall between. Besides the context, it can take two parameters; `min` and `max`. These expresses the values (inclusive) which the `inputPath` context must fall between for the condition to be true. If one of these are omitted, that part of the range will be considered open ended (i.e. if no `max` is specified, any input above or equal to `min` will be considered true)

The `inputPath` can be any context that is expressed as a number. Examples of contexts that can be used with the inRange condition are `http://registry.gpii.net/common/environment/auditory.noise` and `http://registry.gpii.net/common/environment/illuminance`, denoting the background noise in dB and light in illuminance.

**Examples:**

Example 1: Any value of illuminance above or equal to 700 will be considered true.
```{
    "type": "http://registry.gpii.net/conditions/inRange",
    "min": 700,
    "inputPath": "http://registry\\.gpii\\.net/common/environment/illuminance"
}```

Example 2: Any value of illuminance between (incl.) 600 and 1000  will be considered true.
```{
    "type": "http://registry.gpii.net/conditions/inRange",
    "min": 600,
    "max": 1000,
    "inputPath": "http://registry\\.gpii\\.net/common/environment/illuminance"
}```

Example 3: Any value of auditory.noise below or equal to 100 will be considered true.
```{
    "type": "http://registry.gpii.net/conditions/inRange",
    "max": 100,
    "inputPath": "http://registry\\.gpii\\.net/common/environment/auditory\\.noise"
}```

#### http://registry.gpii.net/conditions/timeInRange

This condition is declared using the `http://registry.gpii.net/conditions/timeInRange` type. It is used to express conditions that are related to a timespan. Besides the context, it expects two inputs, both of which much be present. These are `from` and `to` declaring the times which the current time of day must be between (inclusive) for the condition to be evaluated to true. The format of the time should be a string denoting hours (24 hours) and minutes separated by `:`, for example: `"06:30"`, `"19:20"`, `"4:12"`. The condition supports wrapped hours, ie. a `from` time of `"20:00"` and `to` time of `"02:00"`, would evaluate to true between 8 at night and 2 in the morning.

The `inputPath` can be any context that is as a time of day string, matching the format of the `to` and `from` paramaeters (e.g. "19:15"). The one supported by a standard install of the system is `http://registry.gpii.net/common/environment/timeOfDay` denoting the current time of day in the machine's timezone. Note that timezone here is retrieved based on the system time - the geographical location of the user is not taken into account (unless the OS does so by default) - see GPII-1105

**Examples:**

Example 1: Any time between, including, 8 o'clock at night and 8.46 at night will evaluate to true.
```
"conditions": [{
    "type": "http://registry.gpii.net/conditions/timeInRange",
    "from": "20:00",
    "to": "20:46",
    "inputPath": "http://registry\\.gpii\\.net/common/environment/timeOfDay"
}
```

Example 2: Any value between (incl.) 6 at night and 6 in the morning will evaluate to true. (ie. 3 in the afternoon would evaluate to false)
```
"conditions": [{
    "type": "http://registry.gpii.net/conditions/timeInRange",
    "from": "18:00",
    "to": "06:00",
    "inputPath": "http://registry\\.gpii\\.net/common/environment/timeOfDay"
}
```
